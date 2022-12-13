# 2022 - Day 13 -- Distress Signal

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

[https://adventofcode.com/2022/day/13](https://adventofcode.com/2022/day/13)

## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "13", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
[1,1,3,1,1]
[1,1,5,1,1]

[[1],[2,3,4]]
[[1],4]

[9]
[[8,7,6]]

[[4,4],4,4]
[[4,4],4,4,4]

[7,7,7,7]
[7,7,7]

[]
[3]

[[[]]]
[[]]

[1,[2,[3,[4,[5,6,7]]]],8,9]
[1,[2,[3,[4,[5,6,0]]]],8,9]
)
```

```elixir
defmodule Day13 do
  def part_one(input) do
    input
    |> parse()
    |> Enum.map(fn [left, right] -> compare(left, right) end)
    |> Enum.with_index(1)
    |> Enum.filter(fn {bool, _idx} -> bool end)
    |> Enum.map(fn {_, idx} -> idx end)
    |> Enum.sum()
  end

  def part_two(input) do
    sorted_list =
      input
      |> parse()
      |> List.insert_at(0, [[[2]], [[6]]])
      |> Enum.flat_map(& &1)
      |> Enum.sort(fn left, right -> compare(left, right) end)

    {x, y} =
      {Enum.find_index(sorted_list, fn x -> x == [[2]] end),
       Enum.find_index(sorted_list, fn x -> x == [[6]] end)}

    (x + 1) * (y + 1)
  end

  def parse(input) do
    input
    |> String.split("\n\n", trim: true)
    |> Enum.map(&String.split(&1, "\n", trim: true))
    |> Enum.map(fn [left, right] ->
      {left, right} = {Code.string_to_quoted!(left), Code.string_to_quoted!(right)}
      [left, right]
    end)
  end

  def compare(left, left), do: :cont
  def compare(left, right) when is_integer(left) and is_integer(right), do: left < right
  def compare(left, right) when is_integer(left) and is_list(right), do: compare([left], right)
  def compare(left, right) when is_integer(right) and is_list(left), do: compare(left, [right])
  def compare([], _right), do: true
  def compare(_left, []), do: false

  def compare([x | lrest], [y | rrest]) do
    case compare(x, y) do
      :cont -> compare(lrest, rrest)
      bool -> bool
    end
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
[1,1,3,1,1]
[1,1,5,1,1]

[[1],[2,3,4]]
[[1],4]

[9]
[[8,7,6]]

[[4,4],4,4]
[[4,4],4,4,4]

[7,7,7,7]
[7,7,7]

[]
[3]

[[[]]]
[[]]

[1,[2,[3,[4,[5,6,7]]]],8,9]
[1,[2,[3,[4,[5,6,0]]]],8,9]
)

  test "part 1 comparisons" do
    assert Day13.compare([1, 1, 3, 1, 1], [1, 1, 5, 1, 1]) == true
    assert Day13.compare([[1], [2, 3, 4]], [[1], 4]) == true
    assert Day13.compare([9], [[8, 7, 6]]) == false
    assert Day13.compare([[4, 4], 4, 4], [[4, 4], 4, 4, 4]) == true
    assert Day13.compare([7, 7, 7, 7], [7, 7, 7]) == false
    assert Day13.compare([], [3]) == true
    assert Day13.compare([[[]]], [[]]) == false

    assert Day13.compare([1, [2, [3, [4, [5, 6, 7]]]], 8, 9], [1, [2, [3, [4, [5, 6, 0]]]], 8, 9]) ==
             false
  end

  test "part 1" do
    assert Day13.part_one(@example_input) == 13
  end

  test "part 2" do
    assert Day13.part_two(@example_input) == 140
  end
end

ExUnit.run()
```

```elixir
Day13.part_one(puzzle_input)
```

```elixir
Day13.part_two(puzzle_input)
```
