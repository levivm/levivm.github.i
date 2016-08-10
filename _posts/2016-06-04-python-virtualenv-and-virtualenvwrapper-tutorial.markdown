---
title: "Python Virtualenv and Virtualenwrapper Tutorial"
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
jemoji: ':circus_tent:'
---

# Virtual environment

## Virtualenv
Virtualenv is a helpful tool to create isolated Python environments. So, inside those environments you can create your own projects and install it's python packages and dependencies without affecting your system’s site-packages. Also, you can control packages versions for each project and much more.

We are going to need `pip` , you can installing via `easy_install`:


{% highlight raw %}
$ sudo easy_install pip
{% endhighlight %}  

## Installing virtualenv
Install virtualenv via pip:
 
{% highlight raw %}
$ sudo  pip install virtualenv
{% endhighlight %}  

 
We are going to use also __virtualenvwrapper__, a tool to create/delete virtualenvs in an easier way. This  because virtualenv itself is too verbose and a bit complicated. 

Install virtualenvwrapper via pip:
{% highlight raw %}
$ sudo pip install virtualenvwrapper
{% endhighlight %}  

Let's create where our virtual environments would live.

{% highlight raw %}
$ mkdir ~/.virtualenvs
{% endhighlight %}  

You can name it as you want, I rather prefer to use _virtualenvs_.

If you are using standard shell, open your  `~/.bashrc` or `~/.zshrc` if you use oh-my-zsh. Add this two lines

{% highlight raw %} 

export WORKON_HOME=$HOME/.virtualenvs  
source /usr/local/bin/virtualenvwrapper.sh
{% endhighlight %}  

## Starting a new virtual environment

Let's create a virtualenv called _myenv_
{% highlight raw %} 
$ mkvirtualenv myenv
$ > New python executable in myenv/bin/python
$ > Installing setuptools, pip, wheel...done.
(myenv)$
{% endhighlight %}

You will notice a change in your command prompt, it will contain the virtualenv name that indicates that it has been activated.  Now all the python packages that your install without using `sudo` and having your env activated will install inside virtualenv own site-packages.
 

Our new env is located at `~/.virtualenvs/myenv`

## Specifying a python version 

By default, python virtualenv executable is took from python system version, if you want to specify your own version, add `--python=your_python_path`. If you don't know your python path, use `which` command: 

{% highlight raw %} 
$ which python3
$ > usr/local/bin/python3
$ mkvirtualenv --python=usr/local/bin/python3 myenv
{% endhighlight %}


For more information, read virtualenvwrapper [docs](http://virtualenvwrapper.readthedocs.org/en/latest/index.html)

## Shutting down an enviroment
In order to deactivate your virtualenv:
{% highlight raw %}
(myenv)$ deactivate
{% endhighlight %}

## Activating an environment
To activate an existing virtualenv, use command `workon`:


{% highlight raw %}
$ workon myenv
(myenv)$
{% endhighlight %}

## Testing our virtualenv

In order to test our virtualenv, let's install Django.

{% highlight raw %}
(myenv)$ pip install django
> Collecting django
>   Using cached Django-1.10-py2.py3-none-any.whl
> Installing collected packages: django
> Successfully installed django-1.10

{% endhighlight %}

Run a Django command, in this case, let's try to create a new project.

{% highlight raw %} 
(myenv)$ django-admin.py startproject testproject
(myenv)$ ls
> testproject

{% endhighlight %}

Our command just worked as expected. Now we are going to deactivate our virtualenv and try to create a new project.

{% highlight raw %} 
(myenv)$ deactivate
$django-admin.py startproject testproject
> command not found: django-admin.py

{% endhighlight %}

We can be sure that our Django library is only available inside our new virtualenv.

Please, share if you found this post useful :)