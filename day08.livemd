# 2022 - Day 08 -- Treetop Tree House

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

[https://adventofcode.com/2022/day/8](https://adventofcode.com/2022/day/8)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "8", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
30373
25512
65332
33549
35390
)
```

```elixir
defmodule DayEight do
  def part_one(input) do
    trees = parse(input)
    no_cols = length(Enum.at(trees, 0))
    # from left
    trees = mark_visible(trees)

    # from_right
    trees =
      trees
      |> Enum.map(&Enum.reverse(&1))
      |> mark_visible()
      |> Enum.map(&Enum.reverse(&1))

    # from_top
    trees =
      for i <- 0..(no_cols - 1) do
        Enum.map(trees, &Enum.at(&1, i))
      end
      |> mark_visible()

    # from_bottom 
    trees = trees |> Enum.map(&Enum.reverse(&1)) |> mark_visible()

    # count visible
    Enum.map(
      trees,
      fn row ->
        Enum.sum(for {_tree, mark} <- row, do: mark)
      end
    )
    |> Enum.sum()
  end

  def part_two(input) do
    trees = parse2(input)
    {row_no, col_no} = Nx.shape(trees)

    for i <- 1..(row_no - 2) do
      for j <- 1..(col_no - 2) do
        row = trees[[i, ..]] |> Nx.to_flat_list()
        column = trees[[.., j]] |> Nx.to_flat_list()
        visibility_score_in_list(row, j) * visibility_score_in_list(column, i)
      end
    end
    |> List.flatten()
    |> Enum.max()
  end

  def mark_visible_in_list(row) do
    tallest_so_far = 0

    %{marked_row: marked_row} =
      Enum.reduce(
        row,
        %{threshold: tallest_so_far, marked_row: []},
        fn {tree, is_visible}, acc ->
          if tree > acc[:threshold] do
            is_visible = 1
            %{threshold: tree, marked_row: [{tree, is_visible} | acc[:marked_row]]}
          else
            %{acc | marked_row: [{tree, is_visible} | acc[:marked_row]]}
          end
        end
      )

    Enum.reverse(marked_row)
  end

  def mark_visible(trees), do: Enum.map(trees, &mark_visible_in_list(&1))

  def parse(input) do
    input
    |> String.split("\n", trim: true)
    |> Enum.map(fn row ->
      to_charlist(row)
      |> Enum.map(fn hgt -> {hgt, 0} end)
    end)
  end

  def parse2(input) do
    input
    |> String.split("\n", trim: true)
    |> Enum.map(fn row ->
      to_charlist(row)
    end)
    |> Nx.tensor()
    # ?0 == 48
    |> Nx.subtract(48)
  end

  def visibility_score_in_list(list, from) do
    tree = Enum.at(list, from)
    head = Enum.slice(list, 0..(from - 1)) |> Enum.reverse()
    tail = Enum.slice(list, (from + 1)..-1//1)

    visible_before =
      Enum.reduce_while(head, 0, fn neighbor, acc ->
        if neighbor >= tree do
          {:halt, acc + 1}
        else
          {:cont, acc + 1}
        end
      end)

    visible_after =
      Enum.reduce_while(tail, 0, fn neighbor, acc ->
        if neighbor >= tree do
          {:halt, acc + 1}
        else
          {:cont, acc + 1}
        end
      end)

    visible_before * visible_after
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
30373
25512
65332
33549
35390
)

  test "part 1" do
    assert DayEight.part_one(@example_input) == 21
  end

  test "part 2" do
    assert DayEight.part_two(@example_input) == 8
  end
end

ExUnit.run()
```

```elixir
DayEight.part_one(puzzle_input)
```

```elixir
DayEight.part_two(puzzle_input)
```
