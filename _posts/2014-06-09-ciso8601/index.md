---
layout: post
title: 'Introducing ciso8601: A fast ISO8601 date time parser for Python'
date: 2014-06-09
permalink: /posts/ciso8601
author: Thomas Steinacher
thumbnail: ''
metaDescription: ''
tags: [Engineering, Open Source]
---

# Introducing ciso8601: A fast ISO8601 date time parser for Python

**Project link:** [https://github.com/closeio/ciso8601](https://github.com/closeio/ciso8601)

Sometimes it's the little things that make web apps faster. We noticed that loading leads and opportunity view pages in [Close](http://close.com/) wasn't as fast as we wanted it to be in some cases, so we pulled up the profiler. To our surprise, a large portion of time was spent parsing date times, sometimes up to a second.

How did this happen? We serialize certain lead fields in [elasticsearch](http://elasticsearch.org/) so they get returned with our search results. After fetching the fields, we create corresponding instances in our ORM ([see how we made our ORM faster](https://hack.close.com/posts/mongomallard)). Part of this process is converting the ISO8601 date time string into a Python datetime object, which was done using dateutil's parse method:

```python
%timeit dateutil.parser.parse(u'2014-01-09T21:48:00.921000')
10000 loops, best of 3: 111 us per loop
```

On a fast computer it takes over 0.1ms to parse a date time string. For large object structures with thousands of date time objects this can easily add up. For example, each of our leads has two time stamps (date created and updated), and so do our contacts (each lead can have multiple contacts). When serializing many leads this can easily add up to a second.

There are a few ways to solve this problem:

- Refactor our code base so date times are always unserialized lazily (whenever needed)
- Use a faster date time parser

We decided to look for an existing faster date time parser first. After looking for faster parsers, we found [aniso8601](https://bitbucket.org/nielsenb/aniso8601), which was faster but not as fast as we wanted it to be. Other parsers we found were slower, and even Python's datetime module wasn't as fast as date parsing should be. We figured that date time parsing should be fast and that writing a faster date time parser would benefit other projects as well, so we did it! We wrote a C module that parses an ISO8601 date time string and returns a Python date time object.

The above date time is now parsed much faster:

```python
%timeit ciso8601.parse_datetime(u'2014-01-09T21:48:00.921000')
1000000 loops, best of 3: 320 ns per loop
```

Note this is in nano seconds, so that's just 0.00032ms. Now, even if we parse tens of thousands of date time objects, it won't take longer than few ms.

Our profiler confirmed this:

![profiler](./profiler.png)

Check out the project on GitHub: [https://github.com/closeio/ciso8601](https://github.com/closeio/ciso8601)

Contributions welcome!
