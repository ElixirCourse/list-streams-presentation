### Списъци, потоци и рекурсия

Валентин Михов

#HSLIDE

```elixir
[1, :two, "three", %{four: 4}, 'five', [6, [:seven], "eight"]]
```

#HSLIDE

### [head | tail]

```
+-+-+   +-+-+   +-+-+   +-+-+   +-+-+
|1| |-->|2| |-->|3| |-->|4| |-->|5| |
+-+-+   +-+-+   +-+-+   +-+-+   +-+-+
```

#HSLIDE

### Шаблони

```elixir
[a | b] = [1,2,3]
IO.puts a # 1
IO.puts inspect(b) # [2, 3]
```

#HSLIDE

### Обхождане

```elixir
defmodule ListUtils do
  def length([]), do: 0
  def length([_head | tail]), do: 1 + length(tail)
end
```

#HSLIDE

```
ListUtils.length(["cat", "dog", "fish", "horse"])
= 1 + ListUtils.length(["dog", "fish", "horse"])
= 1 + 1 + ListUtils.length(["fish", "horse"])
= 1 + 1 + 1 + ListUtils.length(["horse"])
= 1 + 1 + 1 + 1 + ListUtils.length([])
= 1 + 1 + 1 + 1 + 0
= 4
```

#HSLIDE

### Изграждане

```elixir
defmodule Square do
  def of([]), do: []
  def of([head | tail]), do: [head * head | of(tail)]
end

Square.of([1,2,3,4,5]) # => [1,4,9,16,25]
```

#HSLIDE

```elixir
a = [1,2,3]
b = a ++ [4]
```

vs.

```elixir
a = [1,2,3]
b = [0 | a]
```

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

ListUtils.reduce(["cat", "dog", "horse"], 0, fn _head, acc -> 1 + acc end) # => 3
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

### Сложни шаблони

```elixir
defmodule Meteo do
  def to_fahrenheit([]), do: []
  def to_fahrenheit([%{temperature: temp} = head | tail]) do
    [
      %{head | temperature: c_to_f(temp)} |
      to_fahrenheit(tail)
    ]
  end

  defp c_to_f(celcius) do
    celcius * 1.8 + 32
  end
end
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
  into: %{"fish" => 4},
  do: {x, String.length(x)}
# => %{"cat" => 3, "dog" => 3, "elephant" => 8, "fish" => 4, "mammut" => 6}
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
    # Можете да свалите файла със:
    # curl http://www.gutenberg.org/files/2600/2600-0.txt > war_and_piece.txt
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

:timer.tc(WarAndPiece, :number_of_words_with_enum, [])
# => {1561933, 566309}
:timer.tc(WarAndPiece, :number_of_words_with_stream, [])
# => {833594, 566309}
```

#HSLIDE

### Безкрайни потоци

```elixir
Stream.cycle([1])
|> Enum.take(10)
# => [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
```

#HSLIDE

```elixir
Stream.unfold({0, 1}, fn {a, b} -> {a, {b, a + b}} end) |> Enum.take(10)
# => [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

#HSLIDE

```elixir
defmodule RenderTable do
  def render(list) do
    [
      "<table>\n",
      "  <tr>\n",
      "    <th>Items</th>\n",
      "  </tr>\n",
      render_list(list),
      "</table>"
    ]
  end

  defp render_list(list) do
    Stream.cycle(["odd", "even"])
    |> Enum.zip(list)
    |> Enum.map(&render_item/1)
  end

  defp render_item({class, item}) do
    [
      "<tr>\n",
      "  <td class=\"", class, "\">\n",
      "    ", item, "\n",
      "  </td>\n",
      "</tr>\n"
    ]
  end
end
```

#HSLIDE

### Въпроси?
