# 2022 - Day 05 -- Supply Stacks

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

[https://adventofcode.com/2022/day/5](https://adventofcode.com/2022/day/5)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "5", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
    [D]    
[N] [C]    
[Z] [M] [P]
 1   2   3 

move 1 from 2 to 1
move 3 from 1 to 3
move 2 from 2 to 1
move 1 from 1 to 2
)
```

```elixir
defmodule DayFive do
  def part_one(input) do
    %{stacks: stacks, moves: moves} = parse(input)
    new_stacks = Enum.reduce(moves, stacks, &move9000(&1, &2))

    Enum.sort(new_stacks, fn {i, _}, {j, _} -> i < j end)
    |> Enum.map(fn
      {_, [top | _]} -> top
      _ -> ?_
    end)
    |> to_string()
  end

  def part_two(input) do
    %{stacks: stacks, moves: moves} = parse(input)
    new_stacks = Enum.reduce(moves, stacks, &move9001(&1, &2))

    Enum.sort(new_stacks, fn {i, _}, {j, _} -> i < j end)
    |> Enum.map(fn
      {_, [top | _]} -> top
      _ -> ?_
    end)
    |> to_string()
  end

  def parse(input) do
    input
    |> String.split("\n\n", trim: true)
    |> then(fn [stacks, moves] ->
      {String.split(stacks, "\n", trim: true) |> Enum.reverse() |> Enum.drop(1),
       String.split(moves, "\n", trim: true)}
    end)
    |> then(fn {stack_lines, moves} ->
      %{stacks: parse_stack(stack_lines), moves: parse_moves(moves)}
    end)
  end

  def parse_stack(lines) do
    lines
    |> Enum.map(&to_charlist(&1))
    |> Enum.reverse()
    |> Enum.map(fn line -> line |> Enum.drop(1) |> Enum.take_every(4) end)
    |> Enum.zip()
    |> Enum.map(fn t ->
      Tuple.to_list(t)
      |> Enum.filter(&Regex.match?(~r/[A-Z]/, to_string([&1])))
    end)
    |> Enum.with_index(1)
    |> Enum.reduce(%{}, fn {charlist, idx}, map -> Map.put(map, idx, charlist) end)
  end

  def parse_moves(moves) do
    moves
    |> Enum.map(fn move ->
      Regex.named_captures(~r/move (?<count>\d+) from (?<from>\d+) to (?<to>\d+)/, move)
    end)
  end

  defp move9000(%{"count" => count, "from" => from, "to" => to}, stacks) do
    count = String.to_integer(count)
    from = String.to_integer(from)
    to = String.to_integer(to)

    Enum.reduce(1..count, stacks, fn _, stacks ->
      [crate | tail] = stacks[from]
      to_stack = stacks[to]
      updated_stacks = %{stacks | from => tail, to => [crate | to_stack]}
      updated_stacks
    end)
  end

  defp move9001(%{"count" => count, "from" => from, "to" => to}, stacks) do
    count = String.to_integer(count)
    from = String.to_integer(from)
    to = String.to_integer(to)

    # Enum.reduce(1..count, stacks, fn _, stacks ->
    {crates, tail} = Enum.split(stacks[from], count)
    to_stack = stacks[to]

    %{
      stacks
      | from => tail,
        to => List.flatten([crates, to_stack])
    }
  end
end
```

```elixir
DayFive.parse(example_input)
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
    [D]    
[N] [C]    
[Z] [M] [P]
 1   2   3 

move 1 from 2 to 1
move 3 from 1 to 3
move 2 from 2 to 1
move 1 from 1 to 2
)

  test "part 1" do
    assert DayFive.part_one(@example_input) == "CMZ"
  end

  test "part 2" do
    assert DayFive.part_two(@example_input) == "MCD"
  end
end

ExUnit.run()
```

```elixir
DayFive.part_one(puzzle_input)
```

```elixir
DayFive.part_two(puzzle_input)
```
