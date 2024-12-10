---
layout: post
title:  "Can I run a one-off task from mix?"
date:   2018-06-01 07:20:49 +0200
tags: elixir mix
---
The today's challenge is to run a publish message task from mix.

## First draft without params

After browsing some [documentation][mix_docs_on_hex], i decided for a `task`.

It turns out it is enought to create a `lib/tasks/publish.ex` taks module like:

{% highlight elixir %}
defmodule Mix.Tasks.Publish do
  use Mix.Task

  def run(_) do
    Stomp.publish
  end
end
{% endhighlight %}

where `Stomp.publish` is in `lib/stomp.ex` (repeated from [another post][post_to_activemq]):

{% highlight elixir %}
defmodule Stomp do

  require Logger

  @moduledoc """
  Just send a msg to activemq
  """


  def publish() do
    options = [host: "localhost", port: 61613]
    conn = StompClient.connect(options)

    StompClient.send(conn, "/queue/elixir", "this is a message", %{
      "client-id" => "_client_id",
      "header0" => "just a header",
      "correlation-id" => "ZVv+FpxSDoZ7cWzMf8Dc9N"
    })
    Logger.info("published")
    Process.sleep(1000)
  end
end
{% endhighlight %}

After that, I can invoke my one-off command with:

{% highlight shell %}
$ mix publish

08:45:02.137 [info]  published
{% endhighlight %}



[mix_docs_on_hex]: https://hexdocs.pm/mix/Mix.html
[post_to_activemq]: {% post_url 2018-05-31-can-i-post-to-activemq %}
