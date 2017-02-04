+++
date = "2016-05-20T21:03:05-04:00"
draft = false
title = "django tag for placeholder.it"
tags = ["python", "django"]
categories = ["post"]
readtime = "3"

+++

### The Problem

In the past few weeks I have been using Django to build a website for my local church. The project's early conception required rapid prototyping of the UI, and I found myself using [placeholder images](http://placehold.it) in many places.

```html
<img src="http://placehold.it/350x150"></img>
```

### A Custom Tag

```python
# project/app/templatetags/webextras.py
from django import template

register = template.Library()

@register.simple_tag
def fake_img(width, height):
    return 'https://placehold.it/{}x{}'.format(width, height)
```

For more info on custom tags, read the [Django docs](https://docs.djangoproject.com/en/1.8/howto/custom-template-tags/).

Now, to use the new tag in my templates:

```html
<!-- At top of template -->
{% load fake_img from webextras %}

<!-- ... -->

<img src="{% fake_img 350 150 %}"></img>
<!-- Easily replaced with a static method call later. -->
<img src="{% static 'logo.png' %}"></img>
```