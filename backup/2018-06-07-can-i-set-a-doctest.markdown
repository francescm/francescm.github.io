---
layout: post
title:  "Can I write a doctest?"
date:   2018-06-07 13:20:49 +0200
tags: elixir enum
---
The [doctest][doctest] I'm going to write will test whether the conversion from list to map is correct.

The `publish` task accepts a list. First element in the list is the message paylod; following elements, two by two, will be converted in key, value map pair.

I'm going to write a `Utils` module (`lib/utils.ex`) with a recursive `list_to_map` function:

{% highlight elixir %}
defmodule Utils do
  @moduledoc """
  Utilility functions
"""
  def list_to_map(list, map \\ %{}) do
    if Enum.empty?(list) do
      map
    else
      [key | [value | rest]] = list
      map = Map.put(map, key, value)
      list_to_map(rest, map)
    end
  end
end
{% endhighlight %}

To create a `doctest` just add before functon definition a `@doc` section:
{% highlight elixir %}
[...]
@doc ~S"""
  Turns a list to a map taking elements two by two as key, value.

## Examples

      iex> Utils.list_to_map(["key", "value"])
      %{"key" => "value"}

  """
  def list_to_map(list, map \\ %{}) do
[...]
{% endhighlight %}

In order to test the function with a simple `mix test` a test file is needed (`test/utils_test.exs`):
{% highlight elixir %}
defmodule UtilsTest do
  use ExUnit.Case
  doctest Utils
end
{% endhighlight %}

What about switching to `reduce`?

{% highlight elixir %}
defmodule Utils do
  @moduledoc """
    Utilility functions
  """

  @doc ~S"""
  Turns a list to a map taking elements two by two as key, value.

  ## Examples

      iex> Utils.list_to_map(["key", "value"])
      %{"key" => "value"}

      iex> Utils.list_to_map(["key", "value", "ignore_me"])
      %{"key" => "value"}
  """
  def list_to_map(list) do
    {result, _} =
      Enum.reduce(list, { %{}, nil}, fn el, acc ->
        {map, memory} = acc

        if memory do
          {Map.put(map, memory, el), nil}
        else
          {map, el}
        end
      end)

    result
  end
end
{% endhighlight %}

It is surely better because it can handle the odd-numbered params case, but otherwise it looks less understandable, isn't it?


[doctest]: https://elixir-lang.org/getting-started/mix-otp/docs-tests-and-with.html#doctests
