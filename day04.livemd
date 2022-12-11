# 2022 - Day 04 -- Camp Cleanup

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

[https://adventofcode.com/2022/day/4](https://adventofcode.com/2022/day/4)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "4", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
2-4,6-8
2-3,4-5
5-7,7-9
2-8,3-7
6-6,4-6
2-6,4-8
)
```

```elixir
defmodule DayFour do
  alias MapSet, as: S

  def parse(input) do
    input
    |> String.split("\n", trim: true)
    |> Stream.map(&String.split(&1, ",", trim: true))
  end

  def part_one(input) do
    input
    |> parse()
    |> Stream.map(fn [str1, str2] ->
      {low1, high1} = get_boundaries(str1)
      {low2, high2} = get_boundaries(str2)
      {range1, range2} = {S.new(low1..high1), S.new(low2..high2)}
      overlap = S.intersection(range1, range2)

      if overlap == range1 || overlap == range2 do
        1
      else
        0
      end
    end)
    |> Enum.sum()
  end

  def part_two(input) do
    input
    |> parse()
    |> Stream.map(fn [str1, str2] ->
      {low1, high1} = get_boundaries(str1)
      {low2, high2} = get_boundaries(str2)
      {range1, range2} = {S.new(low1..high1), S.new(low2..high2)}
      overlap = S.intersection(range1, range2)

      if S.size(overlap) > 0 do
        1
      else
        0
      end
    end)
    |> Enum.sum()
  end

  defp get_boundaries(str) do
    Regex.scan(~r/\d+/, str)
    |> then(fn [[low], [high]] ->
      {String.to_integer(low), String.to_integer(high)}
    end)
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
2-4,6-8
2-3,4-5
5-7,7-9
2-8,3-7
6-6,4-6
2-6,4-8
)

  test "part 1" do
    assert DayFour.part_one(@example_input) == 2
  end

  test "part 2" do
    assert DayFour.part_two(@example_input) == 4
  end
end

ExUnit.run()
```

```elixir
DayFour.part_one(puzzle_input)
```

```elixir
DayFour.part_two(puzzle_input)
```
