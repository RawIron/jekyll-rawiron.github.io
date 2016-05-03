---
layout: post
title:  "fsh - A functional programming shell"
date:   2016-05-05 13:12:03 +0200
categories: os
---

GNU core-utils and shell built-in commands are key stones when working with Linux.

* output of commands is space-, tab- or comma-separated format
* filters, aggregates are nicely done with lambda functions and functional programming
* the __|__ of the shell is the __.__ of a functional language
* _sort_ by various columns of core-utils is done with many options

```bash
du -sh . | sort
```

```scala
du(".").sum().filter(humanReadable).sort()
```

[es-shell]: (https://stuff.mit.edu/afs/sipb/user/yandros/doc/es-usenix-winter93.html)
[rc-shell]: (http://doc.cat-v.org/plan_9/4th_edition/papers/rc)