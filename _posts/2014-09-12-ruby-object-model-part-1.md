---
layout: post
title:  "Ruby Object Model, Part 1"
date:   2014-09-12
categories: metaprogramming
---

If you are like me and your first exposure to Ruby was through Ruby on Rails, you may be unfamiliar with many of the underlying mechanics (the "magic") of the language itself. However, there comes a point in your career as a Rails developer where you have to pop the hood and take a look at what is going on inside, either to troubleshoot a complicated issue or to start writing your own modules or engines. In this post, we're going to take a look at Ruby's object model and talk about some of its implications.

Let's say that we have the following code.

{% highlight ruby %}
class Cat
  def meow
    @sound = "meow!"
  end
end
{% endhighlight %}

And then we test this out in irb:

{% highlight bash %}
brutus = Cat.new
brutus.class           # => Cat
{% endhighlight %}

What we are doing here should be fairly familiar. We're defining a Cat class and then initiating a new object of that class, which we are calling brutus. When we run brutus.class, we're running the class() method on the brutus object. The question is, where does this class() method come from? After all, it is not defined inside the Cat class. The only thing we have in there is a meow() method.

Let's explore this a bit further by chaining the superclass() method to what we have.

{% highlight bash %}
brutus.class.superclass     # => Object
{% endhighlight %}

Interesting. So brutus is an instance of the Cat class, which in turn inherits from (or is an) Object. Strange, right? But that is in fact the case in Ruby: everything, including classes, is an object. And Object is where commonly used methods like class() are defined. Since Cat inherits from Object, we can run the class() method on an instance of Cat (i.e. brutus) and get something back (as opposed to an error message).

But wait a minute. If a class is an object, that means every class has to be an instance of something. In other words, if brutus is an instance of Cat, then Cat must be an instance of...

{% highlight bash %}
Cat.class         # => Class
{% endhighlight %}

So the cat class inherits from Object, and is itself an instance of the Class class. The fact that it is an instance of Class is what allows us to use the new() method to initiate a new instance of Cat (which itself does not define the new() method).

But what about Class itself? Let's examine it:

{% highlight bash %}
Class.class            # => Class
{% endhighlight %}

This can be confusing at first. But there's a method to Ruby's madness. In fact, you'll see that what seems like madness right now is actually elegance, and is responsible for some very powerful features of the language. For example, in languages like Java, an instance of a class, MyClass, would simply be a read-only description of the class. In Ruby though, MyClass is quite literally the class itself and can be manipulated like any other object!

This concludes the first half of Ruby's object model. Hopefully it shed some light on some of the mechanisms you have seen in Rails. In Part 2, we will talk about the second half, which includes Module and BasicObject.





