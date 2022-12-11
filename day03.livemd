# 2022 - Day 03 -- Rucksack ReorganizationX

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

[https://adventofcode.com/2022/day/3](https://adventofcode.com/2022/day/3)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "3", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
vJrwpWtwJgWrhcsFMMfFFhFp
jqHRNqRjqzjGDLGLrsFMfFZSrLrFZsSL
PmmdzqPrVvPwwTWBwg
wMqvLMZHhHMvwLHjbvcjnnSBnvTQFn
ttgJtRGJQctTZtZT
CrZsJsPPZsGzwwsLwLmpwMDw
)
```

```elixir
defmodule DayThree do
  require Integer

  def parse(input) do
    input
    |> String.split("\n", trim: true)
    |> Stream.map(&to_charlist(&1))
  end

  def part_one(input) do
    input
    |> parse()
    |> Stream.map(fn charlist ->
      half(charlist)
      |> intersect()
      |> MapSet.to_list()
      |> extract_singleton()
    end)
    |> Stream.map(&priority/1)
    |> Enum.sum()
  end

  def part_two(input) do
    input
    |> parse()
    |> Stream.map(&MapSet.new/1)
    |> Stream.chunk_every(3)
    |> Stream.map(fn sets ->
      intersect(sets)
      |> MapSet.to_list()
      |> extract_singleton()
    end)
    |> Stream.map(&priority/1)
    |> Enum.sum()
  end

  defp half(l) when is_list(l) and Integer.is_even(length(l)) do
    half_len = div(length(l), 2)

    [Enum.take(l, half_len), Enum.take(l, -half_len)]
    |> then(fn [h1, h2] -> [MapSet.new(h1), MapSet.new(h2)] end)
  end

  # Expected to be chars a..z, A..Z 
  defp priority(x) when x >= 97, do: x - 96
  defp priority(x) when x <= 90, do: x - 38

  defp intersect(sets), do: Enum.reduce(sets, &MapSet.intersection(&1, &2))

  defp extract_singleton([x]), do: x
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
vJrwpWtwJgWrhcsFMMfFFhFp
jqHRNqRjqzjGDLGLrsFMfFZSrLrFZsSL
PmmdzqPrVvPwwTWBwg
wMqvLMZHhHMvwLHjbvcjnnSBnvTQFn
ttgJtRGJQctTZtZT
CrZsJsPPZsGzwwsLwLmpwMDw
)

  test "part 1" do
    assert DayThree.part_one(@example_input) == 157
  end

  test "part 2" do
    assert DayThree.part_two(@example_input) == 70
  end
end

ExUnit.run()
```

```elixir
DayThree.part_one(puzzle_input)
```

```elixir
DayThree.part_two(puzzle_input)
```
