---
title: "Python Virtualenv and Virtualenwrapper tutorial"
layout: post
date: 2016-08-10 04:38:13
image: 'http://nvie.com/img/merge-without-ff@2x.png'
description: How to create a python virtualenv using virtualenvwrapper
publish: false
tag:
- python
- virtualenv
- virtualenvwrapper
blog: true
jemoji: ':octocat:'
---

# Virtual environment

## Virtualenv
Virtualenv is a helpful tool to create isolated Python environments. So, inside those environments you can create your own projects and install it's python packages and dependencies without affecting your system’s site-packages. Also, you can control packages versions for each project and much more.

We are going to need `pip`, you can installing via `easy_install`:


{% highlight bash %}

$sudo easy_install pip

{% endhighlight %}