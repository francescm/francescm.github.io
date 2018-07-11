---
layout: post
title:  "Can I add a supervisor?"
date:   2018-06-13 07:54:49 +0200
tags: elixir otp
---
The critical result I want to achieve is to have an application that can restart if ActiveMQ stops or hangs or somehow is not reacheable.

A part of the solution is a `Supervisor`.

But a `Supervisor` can't do nothing unless the supervised apps crashes. And right now, it doesn't happen because the `ConnectionHandler` traps the `on_disconnect` message.

So, the first step is to delete from `lib/stomp/connection_handler.ex` the function:
{% highlight elixir %}
  def handle_info({:stomp_client, :on_disconnect, false}, state) do
    Logger.info("stomp_client connection closed by remote host")
    {:noreply, state}
  end
{% endhighlight %}

Next, we need to add the `Supervisor`, in `lib/stomp/connection_supervisor.ex`:
{% highlight elixir %}
defmodule Stomp.ConnectionSupervisor do
  use Supervisor

  @moduledoc """
  Takes care restarting connection
"""

  def start_link do
    Supervisor.start_link(__MODULE__, [], name: __MODULE__)
  end

  def init([]) do
    connect_opts = [host: "localhost", port: 61613]

    # Define workers and child supervisors to be supervised
    children = [
      # Starts a worker by calling: Worker.start_link(arg1, arg2, arg3)
      # worker(Worker, [arg1, arg2, arg3]),
      worker(Stomp.Connection, [connect_opts]),
    ]

    # See http://elixir-lang.org/docs/stable/elixir/Supervisor.html
    # for other strategies and supported options
    supervise(children, strategy: :one_for_one)
  end

end
{% endhighlight %}

The missing step is the `Connection` module (`lib/stomp/connection.ex`), used to init the connection with the startup actions (subscribe to a queue, hardcoded in code, for now) and using the right callback_hanlder:
{% highlight elixir %}
defmodule Stomp.Connection do
  @moduledoc """
  Connects via Stomp to ActiveMQ
"""
  use GenServer
  require Logger

  def start_link(connect_opts) do
    GenServer.start_link(__MODULE__, connect_opts, name: __MODULE__)
  end

  def init(connect_opts) do
    {:ok, pid} = Stomp.ConnectionHandler.start_link(%{})
    conn = StompClient.connect(connect_opts, callback_handler: pid)
    StompClient.subscribe(conn, "/queue/elixir", id: 1)
    {:ok, %{}}
  end


end
{% endhighlight %}


To test it, add a `start` function to `lib/stomp.ex`:
{% highlight elixir %}
  def start(_type, _args) do
    Stomp.ConnectionSupervisor.start_link
  end
{% endhighlight %}

and try a `mix -S iex` and stop/restart ActiveMQ:
{% highlight iex %}
iex -S mix
[...]
iex(1)>
08:19:26.833 [info]  stomp_client connected: %{"heart-beat" => "0,0", "server" => "ActiveMQ/5.14.5", "session" => "ID:lilypond-49786-1528869096076-3:5", "version" => "1.2"}

08:19:56.314 [error] GenServer #PID<0.151.0> terminating
** (FunctionClauseError) no function clause matching in Stomp.ConnectionHandler.handle_info/2
    (stomp) lib/stomp/connection_handler.ex:19: Stomp.ConnectionHandler.handle_info({:stomp_client, :on_disconnect, false}, nil)
    (stdlib) gen_server.erl:616: :gen_server.try_dispatch/4
    (stdlib) gen_server.erl:686: :gen_server.handle_msg/6
    (stdlib) proc_lib.erl:247: :proc_lib.init_p_do_apply/3
Last message: {:stomp_client, :on_disconnect, false}
State: nil

08:20:06.243 [info]  stomp_client sending SUBSCRIBE: "SUBSCRIBE\nid:1\ndestination:/queue/elixir\nack:auto\n\n\0"

08:20:06.326 [info]  stomp_client connected: %{"heart-beat" => "0,0", "server" => "ActiveMQ/5.14.5", "session" => "ID:lilypond-50434-1528870803016-3:1", "version" => "1.2"}
{% endhighlight %}
