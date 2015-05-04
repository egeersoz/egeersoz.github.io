---
layout: post
title:  "Fun with Strings"
date:   2015-05-03
categories: general
comments: True
---
Phew, long break! The last update was over six months ago! A lot has happened during that time. I quit my job, started my own business, then accepted another job and moved to Austin. Pretty crazy! I've been doing some Ruby code katas during that time, and I figure it might be useful to you, dear readers, if I talk about some of them. A kata is essentially a code challenge that's submitted by users on [Codewars](www.codewars.com). Others then submit their solutions and in the end discuss them.

So in this kata, we'll implement a weirdcase function that takes a single-word string and does the following:

1. Convert the letters with even indexes to uppercase
2. Convert the letters with odd indexes to lowercase

Below are the test cases:

{% highlight ruby %}
describe 'weirdcase' do
  it 'should return the correct value for a single word' do
    Test.assert_equals(weirdcase('This'), 'ThIs');
    Test.assert_equals(weirdcase('is'), 'Is');
    Test.assert_equals(weirdcase('ege'), 'EgE');
  end
end
{% endhighlight %}

Essentially, we need to go through the string letter by letter and decide if we are going to uppercase or lowercase it based on its index.

Here is one way we can do this:

{% highlight ruby %}
def weirdcase(string)
  array = string.split("")
  array.each_with_index do |char, index|
    index.even? ? array[index] = char.upcase : array[index] = char.downcase
  end
  return array.join("")
end
{% endhighlight %}

Here, we're converting the string into an array, then iterating through it using the `each_with_index` iterator, which allows us to check the index of each character during iteration.

That said, there's actually an easier (and cooler) way to do this. Starting with Ruby 1.9, Strings are encoding-aware sequences of characters, so you can simply access their characters by feeding the index as if an array:

{% highlight bash %}
"Test"[1]       # => "e"
{% endhighlight %}

And we can actually iterate through the characters of a string like this:

{% highlight ruby %}
def weirdcase(string)
  string.each_char.with_index do |char, index|
    index.even? ? string[index] = char.upcase : string[index] = char.downcase
  end
  return string
end
{% endhighlight %}

Pretty neat, right? We're modifying the string in-place as opposed to converting it into an array first and then joining the array's characters at the end. I didn't run any performance tests on this but I find it easier to wrap my mind around, personally.

Okay, now suppose an additional requirement is introduced: the string parameters can now contain spaces. How can we account for that?

It's actually not a big deal. Calling `upcase` or `downcase` on a space doesn't do anything, so our code would continue to function as is. But we don't want to be lazy like that! At least I don't. :)

{% highlight ruby %}
def weirdcase(string)
  string.each_char.with_index do |char, index|
    next if char == " "
    index.even? ? string[index] = char.upcase : string[index] = char.downcase
  end
  return string
end
{% endhighlight %}

We're essentially telling it to skip to the next iteration if the character is a space, since there is no need to run `upcase` or `downcase` on it.

We've made pretty good progress so far! Here's one last requirement: if there is more than one word in the parameter string, weirdcase each word separately. To give an example:

{% highlight bash %}
weirdcase("this is a test")       # => "ThIs Is A TeSt"
weirdcase("ruby zen garden")      # => "RuBy ZeN GaRdEn"
{% endhighlight %}

So now we're actually going to do some string-to-array conversions. Specifically, we'll split the string into words first, and then weirdcase each word individually. Here's how:

{% highlight ruby %}
def weirdcase(string)
  word_array = string.split(" ")
  word_array.each_with_index do |word, word_index|
    word.each_char.with_index do |char, char_index|
      char_index.even? ? word[char_index] = char.upcase : word[char_index] = char.downcase
    end
    word_array[word_index] = word
  end
  return word_array.join(" ")
end
{% endhighlight %}

We have two loops here. First, we're looping through the array containing the words in the string. Inside that loop, we have another loop where we're weirdcasing each word, and then replacing the original word in the word array with it. For instance, the word "this" becomes "ThIs". In the end, we're joining the words using a single space.

Of course, Ruby has an incredible number of shorthands and shortcuts and awesome tricks, and it's likely that there are easier ways to write the code above. That's what I like about Codewars. Seeing other people's solutions to the same problem is a great way to learn. For example, here's the top-voted solution for this problem:

{% highlight ruby %}
def weirdcase(string)
  string.gsub(/\w{1,2}/) { |s| s.capitalize }
end
{% endhighlight %}

Wow! It's a very clever solution: characters are extracted from the string in groups of two (which is what the `gsub` with regex '\w{1,2}' does), and then the `capitalize` function capitalizes the first one. This satisfies the problem requirements neatly.

Personally though, I like to be able to understand the code I write if I come back to it later. So I always write things out explicity for the sake of clarity. Besides, overly-creative solutions aren't always a good idea for production because it's easy to make mistakes and forget about edge-cases when trying to be clever.