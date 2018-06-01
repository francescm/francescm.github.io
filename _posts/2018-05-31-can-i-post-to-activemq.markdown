---
layout: post
title:  "Can I publish to activemq with stomp?"
date:   2018-05-31 10:20:49 +0200
categories: elixir, mix, stomp
---
The today's challenge is to push a message to activemq via stomp.

## Mix setup

After creating a project:
{% highlight shell %}
$ mix new stomp
{% endhighlight %}
and after having browsed to find a suitable package to handle stomp (I have choosen [stomp_client][stomp_client]), edit `mix.exs`:

{% highlight elixir %}
  defp deps do
    [
      {:stomp_client, "~> 0.1.1"}
    ]
  end
{% endhighlight %}

You can find the exact line to include from [hex.pm][hex]: just fill `stomp_client` in the `find package` form, than copy the `mix.ex` string.

Next, you need to:
{% highlight shell %}
$ mix deps.get
{% endhighlight %}

Now you can test straight from `iex -S mix`, assuming you have an Apache Activemq broker active on localhost with no security (anonymous access, just the default):

{% highlight elixir %}
Interactive Elixir (1.6.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> options = [host: "localhost", port: 61613]
[host: "localhost", port: 61613]
iex(2)> conn = StompClient.connect(options)
#PID<0.150.0>
iex(3)> StompClient.send(conn, "/queue/elixir", "this is a message", %{"client-id" => "_client_id", "header0" => "just a header", "correlation-id" => "ZVv+FpxSDoZ7cWzMf8Dc9N"})
:ok
{% endhighlight %}

This is really so easy because a `CallBackHandler` is not required.

## Build an app



You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[stomp_client]: https://github.com/miwee/stomp_client
[hex]: https://hex.pm
