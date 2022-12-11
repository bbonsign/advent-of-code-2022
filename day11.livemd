# 2022 - Day 11 -- Monkey in the Middle

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

[https://adventofcode.com/2022/day/11](https://adventofcode.com/2022/day/11)

```elixir
# {:ok, puzzle_input} = KinoAOC.download_puzzle("2022", "11", System.fetch_env!("LB_AOC_COOKIE"))
```

```elixir
puzzle_input = ~s(
Monkey 0:
  Starting items: 59, 65, 86, 56, 74, 57, 56
  Operation: new = old * 17
  Test: divisible by 3
    If true: throw to monkey 3
    If false: throw to monkey 6

Monkey 1:
  Starting items: 63, 83, 50, 63, 56
  Operation: new = old + 2
  Test: divisible by 13
    If true: throw to monkey 3
    If false: throw to monkey 0

Monkey 2:
  Starting items: 93, 79, 74, 55
  Operation: new = old + 1
  Test: divisible by 2
    If true: throw to monkey 0
    If false: throw to monkey 1

Monkey 3:
  Starting items: 86, 61, 67, 88, 94, 69, 56, 91
  Operation: new = old + 7
  Test: divisible by 11
    If true: throw to monkey 6
    If false: throw to monkey 7

Monkey 4:
  Starting items: 76, 50, 51
  Operation: new = old * old
  Test: divisible by 19
    If true: throw to monkey 2
    If false: throw to monkey 5

Monkey 5:
  Starting items: 77, 76
  Operation: new = old + 8
  Test: divisible by 17
    If true: throw to monkey 2
    If false: throw to monkey 1

Monkey 6:
  Starting items: 74
  Operation: new = old * 2
  Test: divisible by 5
    If true: throw to monkey 4
    If false: throw to monkey 7

Monkey 7:
  Starting items: 86, 85, 52, 86, 91, 95
  Operation: new = old + 6
  Test: divisible by 7
    If true: throw to monkey 4
    If false: throw to monkey 5
)
```

## Solution

```elixir
defmodule Q do
  def new(), do: :queue.new()
  def ins(queue, item), do: :queue.in(item, queue)
  def out(queue), do: :queue.out(queue)
  def from_list(queue), do: :queue.from_list(queue)
  def to_list(queue), do: :queue.to_list(queue)
end

defmodule Monkey do
  use GenServer
  # defstruct [:operation, :test, items: Q.new()]

  def start(state, opts) do
    GenServer.start(__MODULE__, state, opts)
  end

  def do_round(name, reduce_factor, worry_factor \\ 1) do
    GenServer.call(name, {:process_items, reduce_factor, worry_factor}, 1000 * 25)
  end

  def get_state(name) do
    GenServer.call(name, :get_state)
  end

  def throw_to(name, item, reduce_factor) do
    GenServer.call(name, {:catch, item, reduce_factor})
  end

  def reset(name) do
    GenServer.call(name, :reset)
  end

  @impl true
  def init(state) do
    {:ok, state}
  end

  @impl true
  def handle_call({:catch, item, reduce_factor}, _from, state = %{items: items, test_no: test_no}) do
    if reduce_factor == 1 do
      {:reply, :ok, %{state | items: Q.ins(items, item)}}
    else
      {:reply, :ok, %{state | items: Q.ins(items, rem(item, reduce_factor))}}
    end
  end

  @impl true
  def handle_call(:get_state, _from, state) do
    {:reply, state, state}
  end

  @impl true
  def handle_call(:reset, _from, state) do
    {:reply, state, %{state | items: Q.new(), inspections: 0}}
  end

  @impl true
  def handle_call({:process_items, reduce_factor, worry_factor}, _from, state) do
    {:ok, count} = process_items(state, reduce_factor, worry_factor)
    {:reply, :ok, %{state | items: Q.new(), inspections: count}}
  end

  defp process_items(
         state = %{operation: op, test: test, items: items, inspections: count},
         reduce_factor,
         worry_factor
       ) do
    case Q.out(items) do
      {{:value, item}, rest} ->
        process_item(op, test, item, reduce_factor, worry_factor)
        process_items(%{state | items: rest, inspections: count + 1}, reduce_factor, worry_factor)

      {:empty, _} ->
        {:ok, count}
    end
  end

  defp process_item(op, test, item, reduce_factor, worry_factor) do
    item = div(op.(item), worry_factor)

    target_monkey = test.(item)
    throw_to(target_monkey, item, reduce_factor)
    {:ok, target_monkey, item}
  end
end
```

```elixir
monkies = [
  Monkey.start(
    %{
      items: Q.from_list([59, 65, 86, 56, 74, 57, 56]),
      inspections: 0,
      operation: fn item -> item * 17 end,
      test_no: 3,
      test: fn item ->
        if Integer.mod(item, 3) == 0 do
          :monkey3
        else
          :monkey6
        end
      end
    },
    name: :monkey0
  ),
  Monkey.start(
    %{
      items: Q.from_list([63, 83, 50, 63, 56]),
      inspections: 0,
      operation: fn item -> item + 2 end,
      test_no: 13,
      test: fn item ->
        if Integer.mod(item, 13) == 0 do
          :monkey3
        else
          :monkey0
        end
      end
    },
    name: :monkey1
  ),
  Monkey.start(
    %{
      items: Q.from_list([93, 79, 74, 55]),
      inspections: 0,
      operation: fn item -> item + 1 end,
      test_no: 2,
      test: fn item ->
        if Integer.mod(item, 2) == 0 do
          :monkey0
        else
          :monkey1
        end
      end
    },
    name: :monkey2
  ),
  Monkey.start(
    %{
      items: Q.from_list([86, 61, 67, 88, 94, 69, 56, 91]),
      inspections: 0,
      operation: fn item -> item + 7 end,
      test_no: 11,
      test: fn item ->
        if Integer.mod(item, 11) == 0 do
          :monkey6
        else
          :monkey7
        end
      end
    },
    name: :monkey3
  ),
  Monkey.start(
    %{
      items: Q.from_list([76, 50, 51]),
      inspections: 0,
      operation: fn item -> item * item end,
      test_no: 19,
      test: fn item ->
        if Integer.mod(item, 19) == 0 do
          :monkey2
        else
          :monkey5
        end
      end
    },
    name: :monkey4
  ),
  Monkey.start(
    %{
      items: Q.from_list([77, 76]),
      inspections: 0,
      operation: fn item -> item + 8 end,
      test_no: 17,
      test: fn item ->
        if Integer.mod(item, 17) == 0 do
          :monkey2
        else
          :monkey1
        end
      end
    },
    name: :monkey5
  ),
  Monkey.start(
    %{
      items: Q.from_list([74]),
      inspections: 0,
      operation: fn item -> item * 2 end,
      test_no: 5,
      test: fn item ->
        if Integer.mod(item, 5) == 0 do
          :monkey4
        else
          :monkey7
        end
      end
    },
    name: :monkey6
  ),
  Monkey.start(
    %{
      items: Q.from_list([86, 85, 52, 86, 91, 95]),
      inspections: 0,
      operation: fn item -> item + 6 end,
      test_no: 7,
      test: fn item ->
        if Integer.mod(item, 7) == 0 do
          :monkey4
        else
          :monkey5
        end
      end
    },
    name: :monkey7
  )
]
```

```elixir
defmodule DayEleven do
  def part_one(monkies) do
    solve(monkies, 20, 3)
  end

  def part_two(monkies) do
    solve(monkies, 10000, 1)
  end

  def solve(monkies, rounds, worry_factor) do
    reduce_factor =
      monkies
      |> Enum.map(fn name -> Monkey.get_state(name).test_no end)
      |> Enum.reduce(1, &(&1 * &2))

    case worry_factor do
      1 -> for _ <- 1..rounds, do: run_round(monkies, reduce_factor, 1)
      _ -> for _ <- 1..rounds, do: run_round(monkies, 1, worry_factor)
    end

    answer =
      monkies
      |> Enum.map(fn monkey ->
        %{inspections: count} = Monkey.get_state(monkey)
        count
      end)
      |> Enum.sort(:desc)
      |> Enum.take(2)
      |> then(fn [x, y] -> x * y end)

    monkies |> Enum.map(&Process.exit(Process.whereis(&1), :kill))

    answer
  end

  def run_round(monkies, reduce_factor, worry_factor) do
    for monkey <- monkies, do: Monkey.do_round(monkey, reduce_factor, worry_factor)
  end
end
```

## Test

```elixir
Monkey.start(
  %{
    items: Q.from_list([79, 98]),
    inspections: 0,
    operation: fn item -> item * 19 end,
    test_no: 23,
    test: fn item ->
      if Integer.mod(item, 23) == 0 do
        :test_monkey2
      else
        :test_monkey3
      end
    end
  },
  name: :test_monkey0
)

Monkey.start(
  %{
    items: Q.from_list([54, 65, 75, 74]),
    inspections: 0,
    operation: fn item -> item + 6 end,
    test_no: 19,
    test: fn item ->
      if Integer.mod(item, 19) == 0 do
        :test_monkey2
      else
        :test_monkey0
      end
    end
  },
  name: :test_monkey1
)

Monkey.start(
  %{
    items: Q.from_list([79, 60, 97]),
    inspections: 0,
    operation: fn item -> item * item end,
    test_no: 13,
    test: fn item ->
      if Integer.mod(item, 13) == 0 do
        :test_monkey1
      else
        :test_monkey3
      end
    end
  },
  name: :test_monkey2
)

Monkey.start(
  %{
    items: Q.from_list([74]),
    inspections: 0,
    operation: fn item -> item + 3 end,
    test_no: 17,
    test: fn item ->
      if Integer.mod(item, 17) == 0 do
        :test_monkey0
      else
        :test_monkey1
      end
    end
  },
  name: :test_monkey3
)

test_input = [:test_monkey0, :test_monkey1, :test_monkey2, :test_monkey3]
```

```elixir
defmodule Test do
  use ExUnit.Case, async: false

  # test "part 1" do
  #   assert DayEleven.solve([:test_monkey0, :test_monkey1, :test_monkey2, :test_monkey3], 20, 3) ==
  #            10605
  # end

  test "part 1 more" do
    assert DayEleven.solve([:test_monkey0, :test_monkey1, :test_monkey2, :test_monkey3], 10000, 1) ==
             2_713_310_158
  end

  # test "part 2" do
  #   assert DayEleven.part_two(@example_input) == 42
  # end
end

ExUnit.run()
```

```elixir
# DayEleven.part_one([
#   :monkey0,
#   :monkey1,
#   :monkey2,
#   :monkey3,
#   :monkey4,
#   :monkey5,
#   :monkey6,
#   :monkey7
# ])
```

```elixir
DayEleven.part_two([
  :monkey0,
  :monkey1,
  :monkey2,
  :monkey3,
  :monkey4,
  :monkey5,
  :monkey6,
  :monkey7
])
```
