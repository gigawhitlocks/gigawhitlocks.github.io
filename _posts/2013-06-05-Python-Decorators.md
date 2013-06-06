---
title: Python Decorators
layout: post
---

Today was my third day at [Hacker School](http://hackerschool.com) in New York City. Now, I haven't been to New York since I was about nine, and I'll have to devote another post at some point to New York because it's really an incredible city. But I digress. Today, not only did I make a number of important commits to my pet project, Cluster, but I also learned about "decorators" in Python.

Now, a Python decorator is "syntactic sugar" for a function that modifies an existing function in a specified way. Now, first of all, what's syntactic sugar?

In this case it means that the `@` character can be used to call a special function above the definition of another function like so:

{% highlight python %}
    @decorator
		def foo():
        pass
{% endhighlight %}

So how do we write a decorator? Well, I'm going to reappropriate the example that was shown to me. Let's say we have a function "meat", defined as such:

{% highlight python %}
    def meat():
        print("bacon")
        print("ham")
{% endhighlight %}

But what if we want to put some bread on that meat and make it a (meaty) sandwhich? First we need to define the function that will become our decorator:

{% highlight python %}
    def bread(contents): 
    # notice that it takes a function as an argument
        def add_bread():
             print("bread")
             contents()
             print("bread")
        return add_bread

    """ note no () because we're 
    returning the function itself, 
    as opposed to returning the 
    value of the called function."""
{% endhighlight %}

This defines a function that takes our first function (meat) and modifies it by defining a new function and returns the new function. Cool!

So right now, we can call meat() and get the following:

{% highlight python %}
    >>> meat()
    bacon
    ham
{% endhighlight %}

If we call bread(meat) we get the following:

{% highlight python %}
    >>> bread(meat)
    bread
    bacon
    ham
    bread
{% endhighlight %}

So what if we always want to call bread(meat) whenever we just call meat()? That's right, now we get to use our decorator. We modify the definition of meat() so that it looks like this:

{% highlight python %}
    @bread
    def meat():
        print("bacon")
        print("ham")
{% endhighlight %}

This is, incidentally, the same as defining meat() normally and then running ```meat = bread(meat)```. After this definition we get the following output:

{% highlight python %}
    >>> meat()
    bread
    bacon
    ham
    bread
{% endhighlight %}


That's pretty cool!
