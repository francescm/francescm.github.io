---
layout: post
title:  "An aside to Enum: generate a connection_id"
date:   2018-06-04 07:20:49 +0200
tags: elixir enum
---
Activemq is happy if a client sets a connection_id. It is also useful when browsing queues from web console because it's like a connection key.

A connection_id is simply a random string.

In order to generate a random string, functional languages use `map` on a list of elements of the expected length. Each element in the list is turned into a random token than the result is joined.

As the working list is just a starting point but the elements are going to be discarded, a good container is a `Range`.

First thing to decide is connection_id length. 
{% highlight elixir %}
char_length = 16
{% endhighlight %}

Next we need to list the allowable symbols (char) that can be found in the connection_id, for example: number and downcased letters.

Again the folder for the allowable char can be a Range. The trick is to build a Range from the ascii values of the chars, but this is easy in Elixir as the ascii value for `a` is `?a`.

Here the `symbol_space`:
{% highlight elixir %}
symbol_space = Enum.concat(?0..?9, ?a..?z)
{% endhighlight %}

I'm going to need ho many elements in List:
{% highlight elixir %}
symbol_size = symbol_space |> to_string |> String.length
{% endhighlight %}

And now connection_id can be generated from with a `map` call:
{% highlight elixir %}
to_string Enum.map((0..char_length), fn(_) -> Enum.at(symbol_space, :rand.uniform(symbol_size - 1)) end)
{% endhighlight %}

Where `:rand.uniform` is an Erlang function to generate an integer up to a given number.
