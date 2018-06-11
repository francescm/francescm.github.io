---
layout: post
title:  "Missing steps to receive messages"
date:   2018-06-11 09:24:49 +0200
tags: elixir otp
---
In order to receive messages (subscribe to a queue) there are some tasks to do:
* write a ConnectionHandler to handle the stomp callback actions;
* write a Supervisor to supervise the ConnectionHandler: nobody wants a activemq restart to kill everything;
* write a MessageHandler to actually do the work. A test MessageHandler could just log the message and `ack` it to the connection. This of course require to modify the subscription factory to remove the auto-ack;
* move connection `options` in a `config` file (this is optional and not strictly related to the message subscrption task).

A stub `ConnectionHandler` can be found in [stomp_client][stomp_client]: it is the `StompClient.DefaultCallbackHandle` module.

Just copy and rename it in file: `lib/stomp/connection_handler.ex`:

{% highlight elixir %}
defmodule ConnectionHandler do
  use GenServer
  require Logger


  @moduledoc """
  Callback handler for stomp calls
"""

  def start_link(initial_state \\ []) do
    GenServer.start_link(__MODULE__, initial_state)
  end

  # GenServer callbacks
  def init(_initial_state) do
    {:ok, nil}
  end

  def handle_info({:stomp_client, :on_connect, message}, state) do
    Logger.info("stomp_client connected: #{inspect(message)}")
    {:noreply, state}
  end

  def handle_info({:stomp_client, :on_connect_error, message}, state) do
    Logger.info("stomp_client connection failure: #{inspect(message)}")
    {:noreply, state}
  end

  def handle_info({:stomp_client, :on_disconnect, true}, state) do
    Logger.info("stomp_client confirmation received for disconnection")
    {:noreply, state}
  end

  def handle_info({:stomp_client, :on_disconnect, false}, state) do
    Logger.info("stomp_client connection closed by remote host")
    {:noreply, state}
  end

  def handle_info({:stomp_client, :on_message, message}, state) do
    Logger.info("stomp_client message received: #{inspect(message, binaries: :as_strings)}")
    {:noreply, state}
  end

  def handle_info({:stomp_client, :on_receipt, receipt_id}, state) do
    Logger.info("stomp_client confirmation received: #{inspect(receipt_id)}")
    {:noreply, state}
  end

  def handle_info({:stomp_client, :on_send, {type, message}}, state) do
    Logger.info("stomp_client sending #{type}: #{inspect(message, binaries: :as_strings)}")
    {:noreply, state}
  end

end
{% endhighlight %}

The module's server part is just the `start_link` function. The client part lists the allowable calls it can receive, sorted by `info` tuple send by stomp_client.

We can test `ConnectionHandler` from `iex`:

{% highlight iex %}
iex -S mix
[...]
iex(1)> {:ok, pid} = ConnectionHandler.start_link(%{})
{:ok, #PID<0.159.0>}
iex(2)> options = [host: "localhost", port: 61613]
[host: "localhost", port: 61613]
iex(3)> c = StompClient.connect(options, callback_handler: pid)
#PID<0.162.0>
iex(4)>
10:02:25.003 [info]  stomp_client connected: %{"heart-beat" => "0,0", "server" => "ActiveMQ/5.14.5", "session" => "ID:lilypond.cesia-usr.unimo.it-50650-1528702246951-3:2", "version" => "1.2"}

nil
iex(5)> StompClient.subscribe(c, "/queue/elixir", id: 1)
:ok

10:03:49.128 [info]  stomp_client sending SUBSCRIBE: "SUBSCRIBE\nid:1\ndestination:/queue/elixir\nack:auto\n\n\0"
iex(6)>
10:03:49.139 [info]  stomp_client message received: %{"body" => "another test", "close_the_door" => "please", "content-length" => "12", "correlation-id" => "zdcnwnk8f82q39j3y", "destination" => "/queue/elixir", "expires" => "0", "message-id" => "ID\\clilypond-50650-1528702246951-3\\c1\\c-1\\c1\\c1", "priority" => "4", "subscription" => "1", "timestamp" => "1528702276938"}

10:04:14.670 [info]  stomp_client message received: %{"body" => "can you see me?", "content-length" => "15", "correlation-id" => "ddsqzww3hkomxswj4", "destination" => "/queue/elixir", "expires" => "0", "message-id" => "ID\\clilypond-50650-1528702246951-3\\c3\\c-1\\c1\\c1", "priority" => "4", "subscription" => "1", "timestamp" => "1528704254666"}

11:21:51.789 [info]  stomp_client connection closed by remote host
{% endhighlight %}

After connecting (by the way, if you didn't want `auto ack`, pass as extra param to `StompClient.subscribe` `ack: "client"`) and receiving two messages, I halted the activemq server; the connection is lost and no further actions can be taken without raising an error.

In order to achieve resilience I think a `Supervisor` is needed.


[stomp_client]: https://github.com/miwee/stomp_client
[doctest]: https://elixir-lang.org/getting-started/mix-otp/docs-tests-and-with.html#doctests
