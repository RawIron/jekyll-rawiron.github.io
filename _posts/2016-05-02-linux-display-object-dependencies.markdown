---
layout: post
title:  "Display object dependencies"
date:   2016-05-02 16:32:12 +0200
categories: code tools
---


The different tools vary regarding

* output
* safety
* gathering

Best known tool is `ldd`.

> Be aware, however, that in some circumstances, some versions of *ldd* may attempt to obtain the dependency information by directly executing the program.

{% highlight bash %}
$ ldd /usr/bin/php

  linux-vdso.so.1 =>  (0x00007fffa3774000)
  libresolv.so.2 => /lib/x86_64-linux-gnu/libresolv.so.2 (0x00007fe0eccf9000)
  libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007fe0ecadf000)
  libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007fe0ec86e000)
  libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fe0ec565000)
  libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fe0ec361000)
  libxml2.so.2 => /usr/lib/x86_64-linux-gnu/libxml2.so.2 (0x00007fe0ebfa6000)
  libssl.so.1.0.0 => /lib/x86_64-linux-gnu/libssl.so.1.0.0 (0x00007fe0ebd3d000)
  libcrypto.so.1.0.0 => /lib/x86_64-linux-gnu/libcrypto.so.1.0.0 (0x00007fe0eb8e2000)
  libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fe0eb518000)
  libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fe0eb2fb000)
  /lib64/ld-linux-x86-64.so.2 (0x000056365a8f2000)
  libicuuc.so.55 => /usr/lib/x86_64-linux-gnu/libicuuc.so.55 (0x00007fe0eaf67000)
  liblzma.so.5 => /lib/x86_64-linux-gnu/liblzma.so.5 (0x00007fe0ead44000)
  libicudata.so.55 => /usr/lib/x86_64-linux-gnu/libicudata.so.55 (0x00007fe0e928d000)
  libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007fe0e8f0a000)
  libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fe0e8cf4000)
{% endhighlight %}

A safer tool is `objdump`.

{% highlight bash %}
$ objdump -p /usr/bin/php | grep NEEDED

  NEEDED               libresolv.so.2
  NEEDED               libz.so.1
  NEEDED               libpcre.so.3
  NEEDED               libm.so.6
  NEEDED               libdl.so.2
  NEEDED               libxml2.so.2
  NEEDED               libssl.so.1.0.0
  NEEDED               libcrypto.so.1.0.0
  NEEDED               libc.so.6
{% endhighlight %}

Another command util is `scanelf`.
On Ubuntu it is part of the `pax-utils` package.

{% highlight bash %}
$ scanelf --needed /usr/bin/php

 TYPE   NEEDED FILE 
ET_DYN libresolv.so.2,libz.so.1,libpcre.so.3,libm.so.6,libdl.so.2,libxml2.so.2,libssl.so.1.0.0,libcrypto.so.1.0.0,libc.so.6 /usr/bin/php
{% endhighlight %}
