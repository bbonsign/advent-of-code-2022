# 2022 - Day 10 -- Cathode-Ray Tube

```elixir
Mix.install([
  {:kino_aoc, git: "https://github.com/ljgago/kino_aoc"},
  {:kino, "~> 0.8.0"},
  {:kino_vega_lite, "~> 0.1.4"},
  {:explorer, "~> 0.4.0"},
  {:nx, "~> 0.2"}
])

alias VegaLite, as: Vl
alias Explorer.DataFrame, as: DF
alias Explorer.Series
ExUnit.start()
```

## Puzzle

[https://adventofcode.com/2022/day/10](https://adventofcode.com/2022/day/10)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "10", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
addx 15
addx -11
addx 6
addx -3
addx 5
addx -1
addx -8
addx 13
addx 4
noop
addx -1
addx 5
addx -1
addx 5
addx -1
addx 5
addx -1
addx 5
addx -1
addx -35
addx 1
addx 24
addx -19
addx 1
addx 16
addx -11
noop
noop
addx 21
addx -15
noop
noop
addx -3
addx 9
addx 1
addx -3
addx 8
addx 1
addx 5
noop
noop
noop
noop
noop
addx -36
noop
addx 1
addx 7
noop
noop
noop
addx 2
addx 6
noop
noop
noop
noop
noop
addx 1
noop
noop
addx 7
addx 1
noop
addx -13
addx 13
addx 7
noop
addx 1
addx -33
noop
noop
noop
addx 2
noop
noop
noop
addx 8
noop
addx -1
addx 2
addx 1
noop
addx 17
addx -9
addx 1
addx 1
addx -3
addx 11
noop
noop
addx 1
noop
addx 1
noop
noop
addx -13
addx -19
addx 1
addx 3
addx 26
addx -30
addx 12
addx -1
addx 3
addx 1
noop
noop
noop
addx -9
addx 18
addx 1
addx 2
noop
noop
addx 9
noop
noop
noop
addx -1
addx 2
addx -37
addx 1
addx 3
noop
addx 15
addx -21
addx 22
addx -6
addx 1
noop
addx 2
addx 1
noop
addx -10
noop
noop
addx 20
addx 1
addx 2
addx 2
addx -6
addx -11
noop
noop
noop
)
```

```elixir
defmodule DayTen do
  @row_len 40
  def part_one(input) do
    register_log =
      input
      |> parse()
      |> Enum.reduce([1], fn instruction, [register | _register_log] = state ->
        register = run_cycle(instruction, register)
        [register | state]
      end)
      |> Enum.reverse()

    [20, 60, 100, 140, 180, 220]
    |> Enum.reduce(0, fn cycle, acc ->
      register = Enum.at(register_log, cycle - 1)
      acc + signal_strength(cycle, register)
    end)
  end

  def part_two(input) do
    frames =
      input
      |> parse()
      |> paint_frames(1)

    IO.puts(Enum.at(frames, 0))
    frames
  end

  def parse(input) do
    input
    |> String.split("\n", trim: true)
    |> Enum.map(fn
      <<"addx ", n::binary>> -> [:start_add, add: String.to_integer(n)]
      "noop" -> :noop
    end)
    |> List.flatten()
  end

  def run_cycle(:start_add, register), do: register
  def run_cycle(:noop, register), do: register
  def run_cycle({:add, n}, register), do: register + n

  def signal_strength(cycle, register), do: cycle * register

  def paint_frames(instructions, init_register) do
    init_state = %{pixels: [], register: init_register}

    %{pixels: pixels_rev} =
      instructions
      |> Enum.with_index(0)
      |> Enum.reduce(init_state, fn {instruction, cycle}, %{pixels: pixels, register: register} ->
        pixel_position = Integer.mod(cycle, @row_len)
        sprite_range = (register - 1)..(register + 1)
        pixel = if(pixel_position in sprite_range, do: ?#, else: ?.)
        register = run_cycle(instruction, register)
        %{pixels: [pixel | pixels], register: register}
      end)

    frames =
      Enum.reverse(pixels_rev)
      |> Enum.chunk_every(240)

    for frame <- frames do
      frame
      |> Enum.chunk_every(@row_len)
      |> Enum.map(&to_string(&1))
      |> Enum.reduce("", &(&2 <> &1 <> "\n"))
      |> String.trim()
    end
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
addx 15
addx -11
addx 6
addx -3
addx 5
addx -1
addx -8
addx 13
addx 4
noop
addx -1
addx 5
addx -1
addx 5
addx -1
addx 5
addx -1
addx 5
addx -1
addx -35
addx 1
addx 24
addx -19
addx 1
addx 16
addx -11
noop
noop
addx 21
addx -15
noop
noop
addx -3
addx 9
addx 1
addx -3
addx 8
addx 1
addx 5
noop
noop
noop
noop
noop
addx -36
noop
addx 1
addx 7
noop
noop
noop
addx 2
addx 6
noop
noop
noop
noop
noop
addx 1
noop
noop
addx 7
addx 1
noop
addx -13
addx 13
addx 7
noop
addx 1
addx -33
noop
noop
noop
addx 2
noop
noop
noop
addx 8
noop
addx -1
addx 2
addx 1
noop
addx 17
addx -9
addx 1
addx 1
addx -3
addx 11
noop
noop
addx 1
noop
addx 1
noop
noop
addx -13
addx -19
addx 1
addx 3
addx 26
addx -30
addx 12
addx -1
addx 3
addx 1
noop
noop
noop
addx -9
addx 18
addx 1
addx 2
noop
noop
addx 9
noop
noop
noop
addx -1
addx 2
addx -37
addx 1
addx 3
noop
addx 15
addx -21
addx 22
addx -6
addx 1
noop
addx 2
addx 1
noop
addx -10
noop
noop
addx 20
addx 1
addx 2
addx 2
addx -6
addx -11
noop
noop
noop
)

  test "part 1" do
    assert DayTen.part_one(@example_input) == 13140
  end

  test "part 2" do
    assert DayTen.part_two(@example_input) ==
             [~s(##..##..##..##..##..##..##..##..##..##..
###...###...###...###...###...###...###.
####....####....####....####....####....
#####.....#####.....#####.....#####.....
######......######......######......####
#######.......#######.......#######.....)]
  end
end

ExUnit.run()
```

```elixir
DayTen.part_one(puzzle_input)
```

```elixir
frames = DayTen.part_two(puzzle_input)

for frame <- frames do
  IO.puts(frame)
end

ZKJFBJFZ
```
