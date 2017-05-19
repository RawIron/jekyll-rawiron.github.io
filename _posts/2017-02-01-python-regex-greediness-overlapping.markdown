---
layout: post
title:  "Python Regex: Greediness and Overlapping-Matches"
date:   2017-02-01 14:39:12 +0200
categories: coding python
---


This is a short summary of thoughts while working on regex challenges.

It is not an extensive, deep dive into the topic.
Also there is no explanation of the inner workings of the regex algorithm given.


Coding is manipulating data. A large part of data are strings.
Most of the time strings need to be sanitized and parsed.

* Sanitizing gets the string ready for the real work on it.
* Parsing extracts the data of interest for the problem at hand.
* Validating is a common operation on extracted words.

The *Python* module **re** contains the implementation of *Python*'s regular expression language.

There is loads of great learning material online or in printed media.
For example the chapter in [Dive into Python3][dive-into-python3] explains the basics and for a comprehensive study the book [Mastering Regular Expressions][mastering-regular-expressions] is a great pick.


## Greediness

For patterns with repetition qualifiers __(*, +, ?, {n,m})__ the match algorithm is greedy.

Search for the first **a** or **b** in the input string and proceed until the first non-matching character is found.

{% highlight python linenos %}
a_or_b = re.compile(r'[ab]+')
pf_groups( a_or_b.search("caabaabcaaa") )

caabaabcaaa
 aabaab
{% endhighlight %}

On *line 4* is the input string and on *line 5* the string matched by the regex.
The match is indented to its position within the input string.


### The programming challenge

Extract words separated by delimiters from a string.

* Word sequences are the match groups, the substrings the regex should extract from the string.
* Delimiter sequences separate the word sequences from each other.

For the case of two words and

>    Z = delimiter sequence, 
>    A = a word sequence,
>    B = a word sequence

it can be expressed as

> ZAZBZ

In the first example the delimiter between those two groups has no known pattern itself.

A trap with greedy repetition qualifiers is to be not specific enough.
It means that one expression in a pattern also matches parts of its following expression.

{% highlight python linenos %}
letters_digits_less = re.compile(r'([a-z]+).*([0-9]+)')
pf_groups( letters_digits_less.search("__abc__123__d") )

__abc__123__d
  abc__123
  abc
         3
{% endhighlight %}

On *line 5* is the match of the regular expression.
*line 6* and *line 7* are the matches of the two groups.
Here the __.\*__ in specific matches __[0-9]*__.
That's why the second group only contains the **3**.

A way to avoid that trap is to specify the *delimiters* as the complement of the second group.

{% highlight python linenos %}
letters_digits_exclude = re.compile(r'([a-z]+)[^0-9]*([0-9]+)')
pf_groups( letters_digits_exclude.search("__abc__123__d") )

__abc__123__d
  abc__123
  abc
       123
{% endhighlight %}

Now *line 6* and *line 7* show the expected result.

In case the *delimiters* have a known pattern things are easier.
The regex for the pattern is placed between the groups.
Note that the delimiter in the following example is optional.

{% highlight python linenos %}
letters_digits_delimiter = re.compile(r'[_+]*([a-z]+)[_+]*([0-9]+)')
letters_digits_delimiter.findall("__abc__123__d")


[('abc', '123')]
{% endhighlight %}


## Overlapping-Matches

The regex algorithm finds the first or all non-overlapping matches.


## Appendix


### notebooks as a tool to write code

There is a [*IPython* notebook][python-notebook-regex-post] in which I toyed with the topic.
In general I highly recommend notebooks to explore a library api or solving one challenge in several different ways.
Or just to clarify your thinking on some topic.

Writing down your thoughts while working on some code is something you definitely wanna try.


### the little helper function
{% highlight python%}

class esc_std_colors:
    OKBLUE = '\033[34m'
    OKGREEN = '\033[32m'
    WARNING = '\033[33m'
    ENDC = '\033[0m'


def pf_group(group, start, end, esc_color=esc_std_colors.OKGREEN):
    print esc_color + group.rjust(end) + esc_std_colors.ENDC 
    
def pf_groups(m):
    print esc_std_colors.OKBLUE + m.string + esc_std_colors.ENDC

    pf_group(m.group(), m.start(), m.end(), esc_std_colors.WARNING)
    for ix, g in enumerate(m.groups()):
        pf_group(g, m.start(ix+1), m.end(ix+1))

{% endhighlight %}



[python-notebook-regex-post]: https://github.com/RawIron/scratch-python/blob/master/gotchas/regex-post.ipynb
[dive-into-python3]: http://www.diveintopython3.net/regular-expressions.html
[mastering-regular-expressions]: http://shop.oreilly.com/product/9781565922570.do