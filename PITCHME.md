### Модулите Enum и Stream

Валентин Михов

#HSLIDE

### Map

```elixir
defmodule ListUtils do
  def map([], _func), do: []

  def map([head | tail], func) do
    [func.(head) | map(tail, func)]
  end
end

ListUtils.map([1,2,3,4,5], fn x -> x * x end) # => [1,4,9,16,25]
```

#HSLIDE

### Reduce

```elixir
defmodule ListUtils do
  def reduce([], acc, _func), do: acc

  def reduce([head | tail], acc, func) do
    reduce(tail, func.(head, acc), func)
  end
end

["cat", "dog", "horse"]
|> ListUtils.reduce(0, fn _head, acc -> 1 + acc end)
# => 3
```

#HSLIDE

```
add_one = fn _head, acc -> 1 + acc end
ListUtils.reduce(["cat", "dog", "horse"], 0, add_one)
= ListUtils.reduce(["dog", "horse"], 1, add_one)
= ListUtils.reduce(["horse"], 2, add_one)
= ListUtils.reduce([], 3, add_one)
= 3
```

#HSLIDE

### Enum module

```elixir
Enum.map([1,2,3,4,5], fn x -> x * x end) # => [1, 4, 9, 16, 25]

Enum.reduce(
  ["cat", "dog", "horse"],
  0,
  fn _head, acc -> 1 + acc end
) # => 3
```

#HSLIDE

### Comprehensions

```elixir
for x <- [1,2,3,4,5], x < 4, do: x * x
# => [1, 4, 9]
```

#HSLIDE

```elixir
for x <- [1,2], y <- [3,4], do: {x, y}
# => [{1, 3}, {1, 4}, {2, 3}, {2, 4}]
```

#HSLIDE

```elixir
for x <- ~w{ cat dog elephant mammut },
  into: %{},
  do: {x, String.length(x)}
# => %{"cat" => 3, "dog" => 3, "elephant" => 8, "mammut" => 6}
```

#HSLIDE

### Потоци

```elixir
1..10_000_000
|> Enum.map(&(&1 + 1))
|> Enum.take(5)
# => [2, 3, 4, 5, 6]
```

vs.

```elixir
1..10_000_000
|> Stream.map(&(&1 + 1))
|> Enum.take(5)
# => [2, 3, 4, 5, 6]
```

#HSLIDE

```elixir
File.read!("binaries.md") # Прочитаме файла
|> String.split("\n") # Превръщаме го във списък от редове
|> Enum.max_by(&String.length/1) # Намираме най-дългият ред
```

#HSLIDE

```elixir
File.stream!("binaries.md")
|> Enum.max_by(&String.length/1)
```

#HSLIDE

```elixir
defmodule WarAndPiece do
  def number_of_words_with_enum do
    File.read!("war_and_piece.txt")
    |> String.split
    |> length
  end

  def number_of_words_with_stream do
    File.stream!("war_and_piece.txt")
    |> Stream.flat_map(&String.split/1)
    |> Enum.reduce(0, fn _, acc -> acc + 1 end)
  end
end
```

#HSLIDE

### Безкрайни потоци

```elixir
Stream.cycle([1, 2])
|> Enum.take(10)
# => [1, 2, 1, 2, 1, 2, 1, 2, 1, 2]
```

#HSLIDE


```elixir
Stream.unfold({0, 1}, fn {current, acc} -> {current, {acc, current + acc}} end)
|> Enum.take(10)
# => [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

#HSLIDE

```elixir
Stream.resource(fn -> File.open!("sample") end,
                fn file ->
                  case IO.read(file, :line) do
                    data when is_binary(data) -> {[data], file}
                    _ -> {:halt, file}
                  end
                end,
                fn file -> File.close(file) end)
```

#HSLIDE

### Data pipelines

```elixir
generate_day_ranges # [{"2018-01-01 00:00:00", "2018-01-01 23:59:59"}, ...]
|> Stream.flat_map(&fetch_prices_for_interval) # Raw price data for every 5 min
|> Stream.flat_map(&convert_to_price_point) # Prepare for import in the DB
|> Store.import
```

See https://github.com/santiment/sanbase2/blob/master/lib/sanbase/external_services/coinmarketcap.ex#L131

#HSLIDE

### HTML rendering example

#HSLIDE

### Въпроси?

![Logo](assets/happy_cat.jpg)
