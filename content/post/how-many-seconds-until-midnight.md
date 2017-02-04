+++
draft = false
date = "2017-01-22T21:19:31-06:00"
title = "how many seconds until midnight?"
tags = ["python", "time", "batman"]
categories = ["post"]
readtime = "4"

+++

### The Problem

<img src="http://i.imgur.com/sivh5Fe.jpg"></img>

> "A countdown of victims that will end at midnight unless our dear Dark Knight stops me first!"
- **Joker** in <a href="http://www.imdb.com/title/tt0519694/">*Holiday Knights*</a>

Imagine you need to know the number of seconds from now until midnight. It could help Batman save Gotham. It could also help a web developer (like me) cache some data in Redis that should self-destruct at the end of the day. How would you do it?

### My Solution

As always, I turn to Python. But it doesn't solve my problem instantly. Python has many ways to deal with date math -- should I use [datetime](https://docs.python.org/3/library/datetime.html#datetime-objects)? [date](https://docs.python.org/3/library/datetime.html#date-objects)? [time](https://docs.python.org/3/library/datetime.html#time-objects)? [timedelta](https://docs.python.org/3/library/datetime.html#timedelta-objects)? An external library like [dateutil](https://dateutil.readthedocs.io/en/stable/)?

Well, for best readability, I decided on a mixture that is pure Python with no external modules:

```python
from datetime import datetime, timedelta

def how_many_seconds_until_midnight():
    """Get the number of seconds until midnight."""
    tomorrow = datetime.now() + timedelta(1)
    midnight = datetime(year=tomorrow.year, month=tomorrow.month, 
                        day=tomorrow.day, hour=0, minute=0, second=0)
    return (midnight - datetime.now()).seconds
```

Or, if you want to remove the timedelta dependency and just use math:

```python
from datetime import datetime

def how_many_seconds_until_midnight():
    """Get the number of seconds until midnight."""
    n = datetime.now()
    return ((24 - n.hour - 1) * 60 * 60) + ((60 - n.minute - 1) * 60) + (60 - n.second)
```
