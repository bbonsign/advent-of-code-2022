# 2022 - Day 12 -- Hill Climbing Algorithm

```elixir
Mix.install([
  {:kino_aoc, git: "https://github.com/ljgago/kino_aoc"},
  {:kino, "~> 0.8.0"},
  {:kino_vega_lite, "~> 0.1.4"},
  {:explorer, "~> 0.4.0"},
  {:nx, "~> 0.2"},
  :libgraph
])

alias VegaLite, as: Vl
alias Explorer.DataFrame, as: DF
alias Explorer.Series
ExUnit.start()
```

## Puzzle

[https://adventofcode.com/2022/day/12](https://adventofcode.com/2022/day/12)

## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "12", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
Sabqponm
abcryxxl
accszExk
acctuvwj
abdefghi
)
```

```elixir
defmodule DayTwelve do
  # backtick char, smaller ?a
  @start ?`
  # one bigger than ?z
  @finish ?{

  def part_one(input) do
    grid = parse(input, "`")
    {start, _} = grid |> Enum.find(fn {_k, v} -> v == @start end)
    {finish, _} = grid |> Enum.find(fn {_k, v} -> v == @finish end)
    path_graph = find_paths(grid)
    shortest_path = path_graph |> Graph.get_shortest_path(start, finish)
    # No of edges in the path
    length(shortest_path) - 1
  end

  def part_two(input) do
    grid = parse(input, "a")
    {finish, _} = grid |> Enum.find(fn {_k, v} -> v == @finish end)
    path_graph = find_paths(grid)
    [start | a_positions] = for {k, ?a} <- grid, do: k
    path_len = Graph.get_shortest_path(path_graph, start, finish) || 40000

    Enum.reduce(a_positions, path_len, fn pos, path_len ->
      next_path = Graph.get_shortest_path(path_graph, pos, finish)

      if next_path do
        if length(next_path) < path_len, do: length(next_path), else: path_len
      else
        path_len
      end
    end)
    |> then(&(&1 - 1))
  end

  def parse(input, replace_S) do
    indexed_input =
      input
      |> String.split("\n", trim: true)
      |> Enum.map(fn string ->
        string |> String.replace("S", replace_S) |> String.replace("E", "{")
      end)
      |> Enum.map(&(to_charlist(&1) |> Enum.with_index()))
      |> Enum.with_index()

    for {row, i} <- indexed_input do
      for({char, j} <- row, do: {{i, j}, char})
    end
    |> List.flatten()
    |> Map.new()
  end

  def neighbors({i, j}), do: [{i - 1, j}, {i + 1, j}, {i, j - 1}, {i, j + 1}]

  def find_paths(grid) do
    paths =
      Enum.reduce(
        grid,
        MapSet.new(),
        fn {pos, elev}, set ->
          Enum.reduce(neighbors(pos), set, fn neighbor, set ->
            case grid[neighbor] do
              nil ->
                set

              neighbor_elev ->
                if neighbor_elev - elev <= 1, do: MapSet.put(set, {pos, neighbor}), else: set
            end
          end)
        end
      )

    Graph.add_edges(Graph.new(), Enum.to_list(paths))
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
Sabqponm
abcryxxl
accszExk
acctuvwj
abdefghi
)

  test "part 1" do
    assert DayTwelve.part_one(@example_input) == 31
  end

  test "part 2" do
    assert DayTwelve.part_two(@example_input) == 29
  end
end

ExUnit.run()
```

```elixir
DayTwelve.part_one(puzzle_input)
```

```elixir
DayTwelve.part_two(puzzle_input)
```
