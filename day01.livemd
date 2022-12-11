# 2022 - Day 01 -- Calorie Counting

```elixir
Mix.install([
  {:kino_aoc, git: "https://github.com/ljgago/kino_aoc"},
  {:kino, "~> 0.7.0"},
  {:kino_vega_lite, "~> 0.1.4"}
])

alias VegaLite, as: Vl
```

## Puzzle

[https://adventofcode.com/2022/day/1](https://adventofcode.com/2022/day/1)



## Solution

```elixir
defmodule DayOne do
  def parse(input) do
    input
    |> String.split("\n\n")
    |> Enum.map(&String.split(&1, "\n", trim: true))
    |> Enum.map(&Enum.map(&1, fn i -> String.to_integer(i) end))
    |> Enum.map(&Enum.sum(&1))
  end

  def part_one(input) do
    input
    |> parse()
    |> Enum.max()
  end

  def part_two(input) do
    parse(input)
    |> Enum.reduce({0, 0, 0}, &update_acc(&1, &2))
    |> Tuple.sum()
  end

  defp update_acc(int, {x, y, z}) when int > x, do: {int, y, z}
  defp update_acc(int, {x, y, z}) when int > y, do: {x, int, z}
  defp update_acc(int, {x, y, z}) when int > z, do: {x, y, int}
  defp update_acc(_, acc), do: acc
end
```

```elixir
DayOne.part_one(puzzle_input)
```

```elixir
DayOne.part_two(puzzle_input)
```
