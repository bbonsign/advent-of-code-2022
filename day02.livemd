# 2022 - Day 02 -- Rock Paper Scissors

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

[https://adventofcode.com/2022/day/2](https://adventofcode.com/2022/day/2)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "2", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
A Y
B X
C Z
)
```

```elixir
defmodule DayTwo do
  # X, Y, Z are interpretted differently between part 1 and 2 
  @hand %{
    "A" => :rock,
    "B" => :paper,
    "C" => :scissors,
    "X" => :rock,
    "Y" => :paper,
    "Z" => :scissors
  }
  @points %{
    rock: 1,
    paper: 2,
    scissors: 3,
    loss: 0,
    tie: 3,
    win: 6
  }

  @goal %{
    "X" => :loss,
    "Y" => :tie,
    "Z" => :win
  }

  def parse(input) do
    input
    |> String.split("\n", trim: true)
    |> Stream.map(&String.split(&1))
  end

  def play_game(hand, hand), do: @points[hand] + @points[:tie]
  def play_game(:rock, :paper = hand), do: @points[hand] + @points[:win]
  def play_game(:rock, :scissors = hand), do: @points[hand] + @points[:loss]
  def play_game(:paper, :rock = hand), do: @points[hand] + @points[:loss]
  def play_game(:paper, :scissors = hand), do: @points[hand] + @points[:win]
  def play_game(:scissors, :rock = hand), do: @points[hand] + @points[:win]
  def play_game(:scissors, :paper = hand), do: @points[hand] + @points[:loss]

  def part_one(input) do
    input
    |> parse()
    |> Stream.map(fn [opponent, me] -> play_game(@hand[opponent], @hand[me]) end)
    |> Enum.sum()
  end

  def apply_strategy(:rock, :loss), do: play_game(:rock, :scissors)
  def apply_strategy(:paper, :loss), do: play_game(:paper, :rock)
  def apply_strategy(:scissors, :loss), do: play_game(:scissors, :paper)
  def apply_strategy(:rock, :tie), do: play_game(:rock, :rock)
  def apply_strategy(:paper, :tie), do: play_game(:paper, :paper)
  def apply_strategy(:scissors, :tie), do: play_game(:scissors, :scissors)
  def apply_strategy(:rock, :win), do: play_game(:rock, :paper)
  def apply_strategy(:paper, :win), do: play_game(:paper, :scissors)
  def apply_strategy(:scissors, :win), do: play_game(:scissors, :rock)

  def part_two(input) do
    input
    |> parse()
    |> Stream.map(fn [opponent, my_goal] ->
      apply_strategy(@hand[opponent], @goal[my_goal])
    end)
    |> Enum.sum()
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
A Y
B X
C Z
)

  test "part 1" do
    assert DayTwo.part_one(@example_input) == 15
  end

  test "part 2" do
    assert DayTwo.part_two(@example_input) == 12
  end
end

ExUnit.run()
```

```elixir
DayTwo.part_one(puzzle_input)
```

```elixir
DayTwo.part_two(puzzle_input)
```
