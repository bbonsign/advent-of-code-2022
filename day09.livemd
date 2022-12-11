# 2022 - Day 09 -- Rope Bridge

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

[https://adventofcode.com/2022/day/9](https://adventofcode.com/2022/day/9)


## Solution

```elixir
{:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "9", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
example_input = ~s(
R 4
U 4
L 3
D 1
R 4
D 1
L 5
R 2
)
example_input2 = ~s(
R 5
U 8
L 8
D 3
R 17
D 10
L 25
U 20
)
```

```elixir
defmodule Segment do
  defstruct head: {0, 0}, tail: {0, 0}

  def update(%Segment{head: {hx, hy}, tail: {tx, ty}}, _head_change = {dx, dy}) do
    h_new = {hxnew, hynew} = {hx + dx, hy + dy}

    case {abs(hxnew - tx), abs(hynew - ty)} do
      {0, 0} ->
        %Segment{head: h_new, tail: {tx, ty}}

      {1, 1} ->
        %Segment{head: h_new, tail: {tx, ty}}

      # head, tail on same row
      {1, 0} ->
        %Segment{head: h_new, tail: {tx, ty}}

      {2, 0} ->
        %Segment{head: h_new, tail: {tx + dx, ty}}

      # head, tail on same column
      {0, 1} ->
        %Segment{head: h_new, tail: {tx, ty}}

      {0, 2} ->
        %Segment{head: h_new, tail: {tx, ty + dy}}

      # head, tail diagonal
      {2, 1} ->
        %Segment{head: h_new, tail: {tx + dx, hynew}}

      {1, 2} ->
        %Segment{head: h_new, tail: {hxnew, ty + dy}}

      {2, 2} ->
        case {abs(hx - tx), abs(hy - ty)} do
          {0, _} ->
            %Segment{
              head: h_new,
              tail: {hxnew, ty + dy}
            }

          {_, 0} ->
            %Segment{
              head: h_new,
              tail: {tx + dx, hynew}
            }

          {1, 1} ->
            %Segment{
              head: h_new,
              tail: {tx + dx, ty + dy}
            }
        end

      x ->
        IO.inspect(x)
        :err
    end
  end
end

defmodule Rope do
  def new() do
    0..8
    |> Enum.map(&("s" <> Integer.to_string(&1)))
    |> Enum.map(&{String.to_atom(&1), %Segment{}})
    |> Keyword.new()
  end

  # end
  def update(rope, change) do
    %{rope: rope} =
      Enum.reduce(
        rope,
        %{rope: rope, change: change},
        fn
          {label, _}, %{rope: rope, change: change} ->
            segment = %{tail: {tx, ty}} = rope[label]
            # tail of this segment is the head of the next segment 
            updated_segment = %{tail: {txnew, tynew}} = Segment.update(segment, change)
            rope = Keyword.update!(rope, label, fn _ -> updated_segment end)
            next_change = {txnew - tx, tynew - ty}
            %{rope: rope, change: next_change}
        end
      )

    rope
  end

  def graph(rope) do
    Vl.new(width: 200, height: 200)
    |> Vl.data_from_values(
      rope
      |> Enum.map(fn {label, %{head: {x, y}}} -> %{x: x, y: y, label: label} end)
    )
    |> Vl.mark(:point, tooltip: true, stroke_width: 4)
    |> Vl.encode_field(:x, "x", type: :quantitative)
    |> Vl.encode_field(:y, "y", type: :quantitative)
    |> Vl.encode_field(:tooltip, "label")
  end
end
```

```elixir
defmodule DayNine do
  def part_one(input) do
    %{visited: visited} =
      input
      |> parse()
      |> Enum.reduce(
        %{segment: %Segment{}, visited: MapSet.new()},
        fn change, %{segment: segment, visited: visited} ->
          new_segment = Segment.update(segment, change)
          %{segment: new_segment, visited: MapSet.put(visited, new_segment.tail)}
        end
      )

    MapSet.size(visited)
  end

  def part_two(input) do
    %{visited: visited, gs: gs} =
      input
      |> parse()
      |> Enum.reduce(
        %{rope: Rope.new(), visited: [], gs: []},
        fn change, %{rope: rope, visited: visited, gs: gs} ->
          new_rope = Rope.update(rope, change)
          # Snapshot of the rope
          g = Rope.graph(new_rope)
          %{rope: new_rope, visited: [new_rope[:s8].tail | visited], gs: [g | gs]}
        end
      )

    # Vl.new()
    # |> Vl.concat(gs |> Enum.reverse(), :vertical)
    Enum.uniq(visited) |> Enum.count()
  end

  def parse(input) do
    input
    |> String.split("\n", trim: true)
    |> Enum.map(&parse_and_expand_steps(&1))
    |> List.flatten()
  end

  def parse_and_expand_steps(step) do
    [dir, number] =
      step
      |> String.split()
      |> then(fn [d, n] -> [d, String.to_integer(n)] end)

    for _ <- 1..number do
      case dir do
        "L" -> {-1, 0}
        "R" -> {1, 0}
        "U" -> {0, 1}
        "D" -> {0, -1}
      end
    end
  end
end
```

## Test

```elixir
defmodule Test do
  use ExUnit.Case, async: true
  @example_input ~s(
R 4
U 4
L 3
D 1
R 4
D 1
L 5
R 2
)
  @example_input2 ~s(
R 5
U 8
L 8
D 3
R 17
D 10
L 25
U 20
)
  test "part 1" do
    assert DayNine.part_one(@example_input) == 13
  end

  test "part 2" do
    assert DayNine.part_two(@example_input) == 1
    assert DayNine.part_two(@example_input2) == 36
  end
end

ExUnit.run()
```

```elixir
DayNine.part_one(puzzle_input)
```

```elixir
DayNine.part_two(puzzle_input)
```

## Plot tails

```elixir
%{visited: visited} =
  puzzle_input
  |> DayNine.parse()
  |> Enum.reduce(
    %{rope: Rope.new(), visited: [], gs: []},
    fn change, %{rope: rope, visited: visited, gs: gs} ->
      new_rope = Rope.update(rope, change)
      # Snapshot of the rope
      g = Rope.graph(new_rope)
      %{rope: new_rope, visited: [new_rope[:s8].tail | visited], gs: [g | gs]}
    end
  )

visited = Enum.uniq(visited) |> Enum.with_index()

Vl.new(width: 600, height: 600)
|> Vl.data_from_values(
  visited
  # |> Enum.with_index()
  # |> then(fn x -> IO.inspect(x) end)
  |> Enum.map(fn {{x, y}, idx} -> %{x: x, y: y, idx: idx, n: "#{idx}: #{x}, #{y}"} end)
)
|> Vl.mark(:point, tooltip: true, stroke_width: 6, size: 10, fill_opacity: 1)
|> Vl.param("grid", select: :interval, bind: :scales)
|> Vl.encode_field(:x, "x", type: :quantitative)
|> Vl.encode_field(:y, "y", type: :quantitative)
|> Vl.encode_field(:tooltip, "n")
|> Vl.encode_field(:color, "idx", type: :quantitative)
```
