---
layout: post
title:  "Understanding Pirate-Speak Using Ruby"
date:   2015-05-05
categories: general
comments: True
---

Another day, another fun and exciting challenge! For this one, we're going to analyze some pirate-speak and try to figure out what they might be saying. Yarrr!

For this scenario, we're parleying with a pirate captain whose ship is about to attack our coastal town. The problem is that he's speaking in anagrams, and we need to figure out, quickly and efficiently, what it is that he is saying. Fortunately, we have a bunch of dictionaries that we can use to decipher their enigmatic language.

Our goal: write a function that takes a string and an array containing the dictionary words, and return an array with elements that are an anagram of that string. If there are no matches, an empty array should be returned.

{% highlight ruby %}
def pirate_speak(anagram, dictionary)
  # code here
end
{% endhighlight %}

Our test cases:

{% highlight ruby %}
Test.expect(pirate_speak("trisf", ["first"]) == ["first"], "Should have found 'first'")
Test.expect(pirate_speak("oob", ["bob", "baobab"]) == [], "Should not have found anything")
Test.expect(pirate_speak("ainstuomn", ["mountains", "hills", "mesa"]) == ["mountains"], "Should have found 'mountains'")
Test.expect(pirate_speak("oolp", ["donkey", "pool", "horse", "loop"]) == ["pool", "loop"], "Should have found 'pool' and 'loop'")
Test.expect(pirate_speak("ortsp", ["sport", "parrot", "ports", "matey"]) == ["sport", "ports"], "Should have found 'sport' and 'ports'")
Test.expect(pirate_speak("ourf", ["one","two","three"]) == [], "Should not have found anything")
{% endhighlight %}

Okay, let's begin.

## The Wrong Way

Most people try to solve anagram problems by counting the number of letters in each word and seeing if the numbers match. For example, the string "ortsp" contains one of each letter, for a total of five letters. And so do the strings from our dictionary, "sport" and "ports". So we might be tempted to write a function that iterates through the original string, creates a hash of each letter where the key is the letter itself and the value is the number of occurrences, and then compares that with a similar hash for each word in the dictionary.

The problem with this approach is that its extremely inefficient, especially for longer words and larger dictionaries. It requires creating a multitude of data structures and then comparing them. This may be OK if we're translating letters written by a pirate (i.e. asyncronous communication), but it might be too slow for a live parley (i.e. real-time translation)!

## A Better Way

When solving problems, it can be useful to get out of the programming context and think about the problem in broader terms. I personally like to draw everything on paper and think about the problem visually. For instance, do we really need to iterate through the anagram and count the number of occurrences for each letter? Since the characters belong to a single alphabet, can't we just re-write each string using a single rule and then compare the result?

Turns out every alphabet has a certain order for its letters, and for anagrams, this comes in very handy. Specifically, the trick is to alphabetically sort both the anagram and each dictionary word and then compare the results. For example:

{% highlight bash %}
"ortsp".chars.sort     #=> ["p", "o", "r", "s", "t"]
"sport".chars.sort     #=> ["p", "o", "r", "s", "t"]
{% endhighlight %}

The resulting arrays are equal, which means that "ortsp" is an anagram of "sport"!

So let's write our function:

{% highlight ruby %}
def pirate_speak(anagram, dictionary)
  final = []
  dictionary.each do |word|
    final << word if word.chars.sort == anagram.chars.sort
  end
  return final
end
{% endhighlight %}

Of course, Ruby is an extremely flexible and expressive language and there's a lot of ways to accomplish the same thing. The above function could be written with just one line of code:

{% highlight ruby %}
def pirate_speak(anagram, dictionary)
  dictionary.select{|w| anagram.chars.sort == w.chars.sort}
end
{% endhighlight %}

`Array:select()` returns a new array containing all elements of the original for which the given block returns a true value. So it's perfect for our use-case.

With a simple and elegant function that can translate pirate speak, we're able to reach an agreement with the pirate captain and the town is saved! Hooray!