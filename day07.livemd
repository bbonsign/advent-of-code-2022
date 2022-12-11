# 2022 - Day 07 -- No Space Left on Device

```elixir
Mix.install([
  {:kino_aoc, git: "https://github.com/ljgago/kino_aoc"},
  {:kino, "~> 0.7.0"},
  {:kino_vega_lite, "~> 0.1.4"}
])

alias VegaLite, as: Vl
ExUnit.start()
```

## Puzzle

[https://adventofcode.com/2022/day/7](https://adventofcode.com/2022/day/7)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "7", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
$ cd /
$ ls
dir a
14848514 b.txt
8504156 c.dat
dir d
$ cd a
$ ls
dir e
29116 f
2557 g
62596 h.lst
$ cd e
$ ls
584 i
$ cd ..
$ cd ..
$ cd d
$ ls
4060174 j
8033020 d.log
5626152 d.ext
7214296 k
)
```

```elixir
defprotocol Size do
  def size(term)
end

defmodule MyFile do
  defstruct [:name, :size]

  defimpl Size do
    def size(%{size: size}), do: size
  end
end

defmodule Dir do
  defstruct [:name, dirs: %{}, files: []]

  def get_subdir(%Dir{dirs: dirs}, name) when is_binary(name) do
    if Map.has_key?(dirs, name) do
      dirs[name]
    else
      {:err, "bad subdir name"}
    end
  end

  def get_subdir(dir, names) when is_list(names) do
    List.foldr(names, dir, fn name, acc -> Dir.get_subdir(acc, name) end)
  end

  def replace_subdir(_dir, [], new_dir) do
    # %{dir | dirs: Map.update(dirs, new_dir.name, new_dir, fn _ -> new_dir end)}
    # {dir, dirs}
    new_dir
  end

  def replace_subdir(dir = %Dir{dirs: dirs}, [subdir_name | rest], new_dir) do
    subdir = get_subdir(dir, subdir_name)
    new_subdir = replace_subdir(subdir, rest, new_dir)
    %{dir | dirs: %{dirs | subdir_name => new_subdir}}
  end

  def update(pwd = %Dir{dirs: dirs}, dir_name) when is_binary(dir_name) do
    if Map.has_key?(dirs, dir_name) do
      pwd
    else
      %{pwd | dirs: Map.put_new(dirs, dir_name, %Dir{name: dir_name})}
    end
  end

  def update(pwd = %Dir{files: files}, %MyFile{} = file) do
    case Enum.find(files, &(&1 == file)) do
      nil -> %{pwd | files: [file | files]}
      _ -> pwd
    end
  end

  defimpl Size do
    def size(%{files: files, dirs: dirs}) do
      files_subtotal =
        files
        |> Enum.map(&Size.size(&1))
        |> Enum.sum()

      dirs_subtotal = dirs |> Enum.map(fn {_, dir} -> Size.size(dir) end) |> Enum.sum()
      files_subtotal + dirs_subtotal
    end
  end
end
```

```elixir
defmodule DaySeven do
  @threshold 100_000
  @total_size 70_000_000
  @free_required 30_000_000

  def part_one(input) do
    root_dir = input |> parse_filetree()

    sum_large_dirs(root_dir, 0)
  end

  def part_two(input) do
    root_dir = input |> parse_filetree()
    root_size = Size.size(root_dir)

    get_dir_sizes(root_dir, %{})
    |> Enum.reduce(root_size, fn {_, size}, acc ->
      if @total_size - (root_size - size) >= @free_required and size < acc do
        size
      else
        acc
      end
    end)
  end

  def get_dir_sizes(dir = %Dir{dirs: dirs}, acc) do
    size = Size.size(dir)
    acc = Map.put_new(acc, dir.name, size)

    Enum.reduce(dirs, acc, fn {_, dir}, acc ->
      Map.merge(acc, get_dir_sizes(dir, acc))
    end)
  end

  def sum_large_dirs(dir = %Dir{dirs: dirs}, acc) do
    size = Size.size(dir)

    large_subdir_total =
      Enum.map(dirs, fn {_, dir} -> sum_large_dirs(dir, acc) end)
      |> Enum.sum()

    if Size.size(dir) < @threshold do
      acc + size + large_subdir_total
    else
      acc + large_subdir_total
    end
  end

  def parse_filetree(input) do
    {ft, _} =
      input
      |> String.split("\n", trim: true)
      |> Enum.reduce({%Dir{name: "/"}, []}, fn line, acc ->
        run_line(line, acc)
      end)

    ft
  end

  # state is tuple: {root_dir, pwd_path}
  # pwd_path is a list: ["pwd_name", "parent_name", ..., "/"]
  defp run_line("$ ls", state), do: state
  defp run_line("$ cd /", _), do: {%Dir{name: "/"}, []}
  defp run_line("$ cd ..", {root, [_pwd | parents]}), do: {root, parents}
  # last $ cd case should be: '$ cd subdir_name'
  defp run_line(<<"$ cd ", _::binary>> = line, {root, pwd}) do
    [_, subdir_name] = Regex.run(~r[\$ cd (.+)], line)
    {root, [subdir_name | pwd]}
  end

  defp run_line(<<"dir ", _::binary>> = line, {root, pwd_path}) do
    [_, dirname] = String.split(line)
    pwd = Dir.get_subdir(root, pwd_path)
    updated_pwd = Dir.update(pwd, dirname)

    if pwd_path == [] do
      {updated_pwd, pwd_path}
    else
      {Dir.replace_subdir(root, Enum.reverse(pwd_path), updated_pwd), pwd_path}
    end
  end

  # last case: line should be a file listing: 'size filename'
  defp run_line(line, {root, pwd_path}) do
    [size_str, name] = String.split(line)
    pwd = Dir.get_subdir(root, pwd_path)
    updated_pwd = Dir.update(pwd, %MyFile{name: name, size: String.to_integer(size_str)})

    if pwd_path == [] do
      {updated_pwd, pwd_path}
    else
      {Dir.replace_subdir(root, Enum.reverse(pwd_path), updated_pwd), pwd_path}
    end
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
$ cd /
$ ls
dir a
14848514 b.txt
8504156 c.dat
dir d
$ cd a
$ ls
dir e
29116 f
2557 g
62596 h.lst
$ cd e
$ ls
584 i
$ cd ..
$ cd ..
$ cd d
$ ls
4060174 j
8033020 d.log
5626152 d.ext
7214296 k  
)

  test "part 1" do
    assert DaySeven.part_one(@example_input) == 95437
  end

  test "part 2" do
    assert DaySeven.part_two(@example_input) == 24_933_642
  end
end

ExUnit.run()
```

```elixir
DaySeven.part_one(puzzle_input)
```

```elixir
DaySeven.part_two(puzzle_input)
```
