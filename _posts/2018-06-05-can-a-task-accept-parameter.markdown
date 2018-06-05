---
layout: post
title:  "Add parameters to a mix task"
date:   2018-06-05 05:55:49 +0200
tags: elixir enum
---
Today's challenge is to add a parameter to a mix task. The `publish` task needs a user-defined payload and maybe also headers.

First strategy is to ask user. It turns out the `Mix.shell.prompt` function is to ask input from user:
{% highlight elixir %}
defmodule Mix.Tasks.Publish do
  use Mix.Task

  @shortdoc "Publish a stomp message"
  def run(_) do
    raw_msg = Mix.shell.prompt "give a message to publish >"
    message = raw_msg |> String.trim
    Mix.shell.info "publishing #{message}"
    Stomp.publish(message)
  end
end
{% endhighlight %}

Another strategy is to accept message from task invocation.

Mix `task` accept a `list` as parameter. If it's needed only message it enough to pick the first element from params list:
{% highlight elixir %}
defmodule Mix.Tasks.Publish do
  use Mix.Task

  @shortdoc "Publish a stomp message"
  def run(params) do
    [ payload | _ ] = params
    Mix.shell.info "publishing #{payload}"
    Stomp.publish(payload)
  end
end
{% endhighlight %}

To handle more user-defined headers we would need some recursion, but it would be even better to enjoy a reduce pattern.
