---
layout: post
title:  "Ruby Object Model, Part 2"
date:   2014-09-20
categories: metaprogramming
comments: True
---

Welcome to Part 2 of the Ruby Object Model. In [Part 1]({% post_url 2014-09-12-ruby-object-model-part-1 %}), we found that:

* All classes are instances of the Class class, including Class itself.
* All classes inherit from the Object class.

In this post, we'll continue our exploration and look at Modules and something called the BasicObject.

We already saw that:

{% highlight bash %}
Class.class            # => Class
{% endhighlight %}

But this simply shows that the Class class points to itself, which sort of makes sense. What does it inherit from though?

{% highlight bash %}
Class.superclass            # => Module
{% endhighlight %}

If you have been programming Rails for a while, you'll be familiar the term Module. The result of the above statement means that every class is also a module. A class simply has three additional instance methods (new, superclass and allocate) that allow you to create objects of that class and arrange your classes into hierarchies (e.g. "class Cat < Animal" means Cat is a subclass of Animal).

Technically speaking, classes and modules can be used interchangably. However, as a best practice, you should use a class when you intend it to be instantiated or inherited, and a module when you intend it to be included (i.e. to "extend" another class or module). The more explicit your intentions, the easier your code will be to read, not just by other people but also your future self!

Another type of scenario where Modules come in handy is when you need to create a namespace. Suppose your project uses a Zoo gem that already has the Cat class defined, but you want to have your own Cat class without monkeypatching the one in the gem. What you can do is wrap your class in a module:

{% highlight ruby %}
module Wildlife
  class Cat
    def meow
      @sound = "ROAR!"
    end
  end
end
{% endhighlight %}

Then, you can refer to your own Cat class using double colons:

{% highlight bash %}
cheetah = Wildlife::Cat.new
cheetah.meow         # => "ROAR!"
{% endhighlight %}

As mentioned previously, all classes are objects. This holds true for modules:

{% highlight bash %}
Module.class                 # => Class
Module.superclass            # => Object
{% endhighlight %}

Object is the default root of all Ruby objects. It inherits from BasicObject, but in my opinion that's not terribly important because BasicObject is just an empty class. You only need to pay attention to it if you decide to create object hierarchies that are independent from the one Ruby provides.

What is important about the Object class is that it includes the Kernel module, which contains the definitions of numerous important methods. You can take a look at these by doing:

{% highlight bash %}
Kernel.methods                 # => [...]
Kernel.instance_methods        # => [...]
{% endhighlight %}

In Ruby 2.1.1 (which I'm using), there are 155 class methods and 47 instance methods in the Kernel module. The class methods include methods such as puts, print, require, lambda, eval, and many, many more. You can take a look at the class methods [here](http://ruby-doc.org/core-2.1.2/Kernel.html).

This concludes our brief exploration of the Ruby Object Model. To summarize, here's a diagram from Paola Perrotta's excellent book, Metaprogramming in Ruby 2:

<img src="/images/Ruby_object_model.png">

The boxes denote the entities and the arrows denote the relationships between them. BasicObject and Kernel are not show, but at this point it shouldn't be too difficult to figure out where they fit into the picture!
