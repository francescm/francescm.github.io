---
layout: post
title:  "Can I wire a GenServer to the subscription framework?"
date:   2018-06-18 06:54:49 +0200
tags: elixir otp
---
When a GenServer is started via `start_link` it is given an "initial state". The initial state is then included in all return message, along with the status symbol. Failure to do so means to lose the state.

The today's task is to allow a `MessageHandler` GenServer to ack a message recevived by subscripting a queue from the `ConnectionHandler` server.

The challenge is how to wire toghether these blocks: the matter is there are a lot of actors involved:
* Connection: a orchestrator;
* StompClient: the actual workhorse;
* ConnectionHandler: the callback module that handles the StompClient calls;
* ConnectionSupervisor: a supervisor to add resilience;
* MessageHandler: the piece to fit in the puzzle: it is the hyphotetical component that processes the message. In this test setup it is enough it acks it.

From `mix.exs`, the `Stomp` module is called. From is `start` method, `Stomp.ConnectionSupervisor` is activated, which, in turn, launches a supervised `Connection`.

`Stomp.MessageHander`, to ack a message, needs a hold to `StompClient`. On the other hand, `Stomp.ConnectionHandler` needs a way to invoke `Stomp.MessageHander`.

This is the `init` method in `Stomp.Connection` that allows this all:
{% highlight elixir %}
  def init(connect_opts) do
    {:ok, pid} = Stomp.ConnectionHandler.start_link(%{})
    conn = StompClient.connect(connect_opts, callback_handler: pid)
    {:ok, msg_handler} = Stomp.MessageHandler.start_link(%{conn: conn})
    Stomp.ConnectionHandler.register(pid, msg_handler)
    StompClient.subscribe(conn, "/queue/elixir", id: 1, ack: "client")
    {:ok, %{}}
  end
{% endhighlight %}

`Stomp.ConnectionHandler` receives the link to `Stomp.MessageHandler` throught a call by `register` (the call to `Stomp.MessageHandler` is in the :on_message hadler):

{% highlight elixir %}
  # client part
  def register(conn, msg_handler) do
    Logger.info("registering: #{inspect msg_handler}")
    GenServer.cast(conn, {:register, msg_handler})
  end

  # server part
  def handle_cast({:register, msg_handler}, _) do
    Logger.info("doing register for #{inspect msg_handler}")
    {:noreply, %{msg_handler: msg_handler}}
  end

  def handle_info({:stomp_client, :on_message, message}, state) do
    %{msg_handler: msg_handler} = state
    Logger.info("stomp_client message received: #{inspect(message, binaries: :as_strings)}")
    Stomp.MessageHandler.process(msg_handler, message)
    {:noreply, state}
  end
{% endhighlight %}

Despite this direct wiring works, a registry looks a useful add-on.
