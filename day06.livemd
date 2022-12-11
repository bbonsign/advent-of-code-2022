# 2022 - Day 06 -- Tuning Trouble

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

[https://adventofcode.com/2022/day/6](https://adventofcode.com/2022/day/6)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "6", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(mjqjpqmgbljsphdztnvjfqwrcgsmlb)
```

```elixir
defmodule DaySix do
  def part_one(input), do: find_marker_index(input, 4)

  def part_two(input), do: find_marker_index(input, 14)

  defp find_marker_index(input, marker_size) do
    [{_, idx}] =
      parse(input)
      |> Stream.chunk_every(marker_size, 1)
      |> Stream.with_index(marker_size + 1)
      |> Stream.take_while(fn {cl, _} -> MapSet.size(MapSet.new(cl)) < marker_size end)
      |> Enum.take(-1)

    idx
  end

  def parse(input) do
    to_charlist(input)
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(mjqjpqmgbljsphdztnvjfqwrcgsmlb)

  test "part 1" do
    assert DaySix.part_one(@example_input) == 7
    assert DaySix.part_one("bvwbjplbgvbhsrlpgdmjqwftvncz") == 5
    assert DaySix.part_one("nppdvjthqldpwncqszvftbrmjlhg") == 6
    assert DaySix.part_one("nznrnfrfntjfmvfwmzdfjlvtqnbhcprsg") == 10
    assert DaySix.part_one("zcfzfwzzqfrljwzlrfnpqdbhtmscgvjw") == 11
  end

  test "part 2" do
    assert DaySix.part_two(@example_input) == 19
    assert DaySix.part_two("bvwbjplbgvbhsrlpgdmjqwftvncz") == 23
    assert DaySix.part_two("nppdvjthqldpwncqszvftbrmjlhg") == 23
    assert DaySix.part_two("nznrnfrfntjfmvfwmzdfjlvtqnbhcprsg") == 29
    assert DaySix.part_two("zcfzfwzzqfrljwzlrfnpqdbhtmscgvjw") == 26
  end
end

ExUnit.run()
```

```elixir
DaySix.part_one(puzzle_input)
```

```elixir
DaySix.part_two(puzzle_input)
```
