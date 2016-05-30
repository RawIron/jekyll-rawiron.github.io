---
layout: post
title:  "A Data Wrangler's view on the GNU core-utils"
date:   2016-05-27 13:12:03 +0200
categories: os linux zsh
---


### Data Wrangler's view

GNU core-utils combined with the shell built-ins are the bread and butter when working interactively with Linux. 

The *pipe* that sends byte streams from `stdout` of one command to `stdin` of another command is a powerful mechanism.
The main idea is to combine many simple tools to solve a given task.
A single command-line has a concise syntax and reads nicely.

{% highlight bash %}
cat my.txt | grep -i "student" | wc -l
{% endhighlight %}

It is a great toolbox, hands down.
In case a certain command-line is needed frequently an `alias` can take care of it.
If some automation would help then a short shell script will do.
And of course there are the nice extras which *oh-my-zsh* provides.

Switching frequently between command-line work and one of the *REPL* tools is daily routine for a Data Wrangler.
For the Data Wranglers using Python `ipython` (`jupyter`) is a nice *REPL* (the use of the term *REPL* is a bit casual here).

{% highlight python %}
In [1]: len(filter(lambda x: True if x.find("student") > 0 else False, open("my.txt").readlines()))
Out[1]: 17
{% endhighlight %}

The similarities between the previous (`zsh` and core-utils) and the current tool (`jupyter`) are obvious.

* The work is interactive and often different tools are combined in several iterations until a satisfactory result is found.
* The processing model in both cases can be shortly described as "applying multiple functions consecutively on (a stream of) data".

{% highlight python %}
function_f | function_g
# can be written as
f() | g()
# mathematical notation
f(g())
# functional syntax
f().g()
{% endhighlight %}


### Categorizing the core-utils

Probably a quick categorization of the [GNU core-utils][core-utils] can help.
And creates more opportunities to do some data wrangling.

#### So how many core-utils are there? 
To answer that one the core-utils can be *cloned* from a git repo.
Then use some of the core-utils to do the counting.
The first iteration works but comes with an ugly output on `stderr`

{% highlight bash %}
ls -1 *.c | cut -f 1 -d '.' | xargs man -f -s 1 | grep -v 'nothing appropriate' | wc -l
{% endhighlight %}

Removing the output on `stderr` and only the number of core-utils is displayed.
Altogether 96 core-utils.

{% highlight bash %}
$ ls -1 *.c | cut -f 1 -d '.' | xargs man -f -s 1 2>/dev/null | wc -l               
96
{% endhighlight %}

96 is small enough to manually categorize them.

#### core-util name plus short description
Next step is to dump the name of every core-util with its short description into a file.
Just by the quick description `cut` seems like the perfect tool.
`cut` is pretty much the *project* operation in Relational Algebra or the *select* clause in SQL.

The output of `man -f` is not great.
The words in the short description use the same delimiter (space) which is used to separate some of the columns.
Actually space and '-' are used as delimiters.
Some core-utils prefer a human-friendly output format.

{% highlight bash %}
$ ls -1 *.c | cut -f 1 -d '.' | xargs man -f -s 1 2>/dev/null

base64 (1) - base64 encode/decode data and print to standard output
basename (1) - strip directory and suffix from filenames
...             
{% endhighlight %}

That's not a roadblock.
Luckily the description is in the "last column".

{% highlight bash %}
$ ls -1 *.c | cut -f 1 -d '.' | xargs man -f -s 1 2>/dev/null | tr -s ' ' | cut -d ' ' -f 1,3,4-

base64 - base64 encode/decode data and print to standard output
basename - strip directory and suffix from filenames
...
{% endhighlight %}

That's more like the output for the data-wranglers among the command-line users.


#### Categorize manually
Now back to the categorization of the core-utils.
In the output file three new columns got manually added.

* pure or side-effect
* type of data processing
* type of node in processing pipeline

The definitions of the data processing types are as follows:

* filter - input rows but less of them
* sort - input rows in sorted order
* transform - input rows with modified content
* reduce - input rows get aggregated; new and less rows as result
* expand - input rows get discarded; new and more rows as result
* destroy - input rows get discarded; new and less rows as result
* copy - input rows as result; can be in different order
* map - must be called with `xargs`

There are two oddities here.
The operation `destroy` is defined but no `create`.
And the `map` does not fit in.
It defines how a core-util must be called in the data pipeline while all the other terms describe what they do on the input data.

The definitions of the node types:

* source - does not read input (no __\|__ to the left)
* sink - does not write output (no __\|__ to the right)
* step - reads input and writes output

The complete result of the manual categorization is wrapped up in this [file]({{ site.github.url }}/files/core-utils.txt).

All the core-utils are looked at as if they were developed to be used in a data processing pipeline.
The existence of shell-scripts and the command-line is ignored.

As an example the core-util `sleep` would be oddly used as follows:

{% highlight bash %}
echo 2 8 | xargs sleep
{% endhighlight %}

And of course the most common idiom becomes cluttered.

{% highlight bash %}
echo 2 | xargs sleep
# instead of simply "sleep 2"
{% endhighlight %}

But again for the purpose of this categorization it is fine.


### Learned so far

* output of commands is space-, tab- or comma-separated format
* filters, aggregates are nicely done with lambda functions and functional programming
* the __\|__ of the shell is the __.__ of a functional language
* _sort_ by various columns of core-utils is done with many options
* `awk` is the anonymous function or lambda
* `xargs` is the map function

* The __\|__ cannot fork and join. Processing is always sequential.
Any parallel processing is hand-coded using for example `split`, `join`.
* Many commands create no output (no news are good news).


### Future Work

{% highlight bash %}
du -sh . | sort
{% endhighlight %}

{% highlight scala %}
du(".").sum().filter(humanReadable).sort()
{% endhighlight %}

{% highlight bash %}
cat -n 2 my.txt | wc
{% endhighlight %}

{% highlight python %}
cat("my.txt").n(2).end().pipe().wc.end()
{% endhighlight %}

[core-utils]: http://www.gnu.org/software/coreutils/coreutils.html
[es-shell]: https://stuff.mit.edu/afs/sipb/user/yandros/doc/es-usenix-winter93.html
[rc-shell]: http://doc.cat-v.org/plan_9/4th_edition/papers/rc
