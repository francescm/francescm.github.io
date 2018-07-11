---
layout: post
title:  "Can I understand registry?"
date:   2018-06-21 06:09:49 +0200
tags: elixir otp
---
A [Registry][Registry] works as a catalog for server processes. It allows to:
* call them without using the pid and;
* dinamically register/deregister processes.

Out-of-the-box a Registry can be used through the `:via` [GenServer][GenServer]'s Name Registration:

{% highlight elixir %}
Interactive Elixir (1.6.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>  {:ok, _} = Registry.start_link(keys: :unique, name: MyRegistry)
{:ok, #PID<0.86.0>}
iex(2)> {:ok, _} = Agent.start_link(fn -> 0 end, name: {:via, Registry, {MyRegistry, "zero_agent"}})
{:ok, #PID<0.89.0>}
iex(3)> Registry.whereis_name({MyRegistry, "room1"})
:undefined
iex(4)> Registry.whereis_name({MyRegistry, "zero_agent"})
#PID<0.89.0>
{% endhighlight %}

The Name Registration process allows us to name a process with the `via tuple` way: the first token of the tuple is the fixed atom `:via`, next follows the Module name of the Registry, next the actual command to give the name.
 
The today's challenge is to understand why:
{% highlight elixir %}
Registry.whereis_name({MyRegistry, "zero_agent"}) |> Process.exit(:kill)
{% endhighlight %}
kills the Registry as well.

[Registry]: https://hexdocs.pm/elixir/Registry.html
[GenServer]: https://hexdocs.pm/elixir/GenServer.html
