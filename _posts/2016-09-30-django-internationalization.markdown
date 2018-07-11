---
title: "Django App Internationalization"
layout: post
date: 2016-09-30 04:38:13
image: 'http://practicalcoding.in/check/images/fulls/b3.png'
description: Learn how you can set up your Django app in order to be available in several languages.
tag:
- Django
- Python
- Translation
- Internationalization
- i18n
blog: true
author: "Levi Velázquez"

---


## Internationalization

Many time we face the problem to build our apps in away that theirself be avaiable in several languages. That is what we call, __Internationalization__. 

Let's see how we can archive this. 

## Django Translations

Before starting, let's clone our base code (cat-quotes), setup virtualenv and initialize the project.

{% highlight bash %}
python -m venv cat-quotes-project # Creating our virtualenv
cd cat-quotes-project
source bin/activate
git clone git@github.com:levivm/cat-quotes.git cat-quotes
cd cat-quotes
pip install -r requirements.txt
./manage.py migrate
{% endhighlight %}

This should be our dir estructure

{% highlight bash %}
cat
├── cat
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── db.sqlite3
├── manage.py
└── quotes
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── migrations
    │   └── __init__.py
    ├── models.py
    ├── templates
    │   └── quotes.html
    ├── tests.py
    ├── urls.py
    └── views.py
{% endhighlight %}

Run your server `./manage.py runserver 8000` and visit `http://localhost:8000/cats/quotes/`. You should view something like this

{% include image.html path="posts/screen_1.png"
                      path-detail="posts/screen_2.png"
                      alt="Sample image" %}

There two main steps in order to translate a django app. The first one is marking the strings that should be translated and second one is to generate language files where you would define the translated strings for those marked ones. 


### Marking strings.

Django provides two function for defining translations strings. They are `ugettext()` and `ugettext_lazy()`. 


We are going to use `ugettext_lazy()`. So, we need to start marking. 

Let's find our cat quotes, open `quotes/views.py`


{% highlight python %}
from django.views.generic import TemplateView


class CatQuotesView(TemplateView):
    template_name = "quotes.html"

    def get_context_data(self, *args, **kwargs):
        context = super(CatQuotesView, self).get_context_data(**kwargs)

        cat_quote_one = "I'm bilingual cat and know how to meow in several languages: Meow."
        cat_quote_two = "I'm not angry, go to kill yourself."
        context.update({
            'cat_quote_one': cat_quote_one,
            'cat_quote_two': cat_quote_two
        })
        return context
{% endhighlight %} 

Import `ugettext_lazy`

{% highlight python %}
from django.utils.translation import ugettext_lazy as _ 
{% endhighlight %}

Wrap our cat quotes using `_()`. Don't worry, `_` is just an alias, you can use `ugettext_lazy()` if you want.


{% highlight python %}
cat_quote_one = _("I'm bilingual cat and know how to meow in several languages: Meow.")
cat_quote_two = _("I'm not angry, go to kill yourself.")
{% endhighlight %}

So, our view ends like this

{% highlight python %}
from django.views.generic import TemplateView
from django.utils.translation import ugettext_lazy as _ 


class CatQuotesView(TemplateView):
    template_name = "quotes.html"

    def get_context_data(self, *args, **kwargs):
        context = super(CatQuotesView, self).get_context_data(**kwargs)

        cat_quote_one = _("I'm bilingual cat and know how to meow in several languages: Meow.")
        cat_quote_two = _("I'm not angry, go to kill yourself.")
        context.update({
            'cat_quote_one': cat_quote_one,
            'cat_quote_two': cat_quote_two
        })
        return context
{% endhighlight %}

### Translating marked strings or translation strings.


Before starting to allow string translation, we need to create a folder named `locale` within our `quotes` app. In the same way, if you have more apps, you just need to create it for each one that you want to translated. In the `locale` dir is where are going to live all our translation files.  


Let's to create a message file for our translation strings. That file is a text-plain representing a single language. It contains all available translation strings(Our marked strings) and how they should be translated. Those files have a **.po** file extension. 

To create or update our message file, we are going to use command `makemessages` provided by Django. 

Note: We need to [install _GNU gettext toolset_](https://stackoverflow.com/questions/27220052/django-i18n-make-sure-you-have-gnu-gettext-tools)

To create or update a message file, we need to run the command on the root of our app containing marked strings. In our case `quotes/`.

{% highlight bash %}
django-admin makemessages -l es_ES
{% endhighlight %}

Where **es_ES** represents the local name for the message file. For example, **en-US** for United State English, **de** for German and so on.

It would recollect all our marked strings and update the `.po`  file. Then, we need to open our .po file and set the correct translation for each string. 

After running the command, your `quotes` app  directory should looks like this:

{% highlight raw %}
├── __init__.py
├── admin.py
├── apps.py
├── locale
│   └── es-ES
│       └── LC_MESSAGES
│           └── django.po
├── migrations
│   └── __init__.py
├── models.py
├── templates
│   └── quotes.html
├── tests.py
├── urls.py
└── views.py
{% endhighlight %}

We got a new folder named `es-ES` (Folder for Spanish translation files), within it, there are a folder LC_MESSAGES containing a `.po` file. That file should has entries like this:

{% highlight python %}
#: views.py:11
msgid "I'm bilingual cat and know how to meow in several languages: Meow."
msgstr ""

#: views.py:12
msgid "I'm not angry, go to kill yourself."
msgstr ""
{% endhighlight %}

`msgid` is the translation string, which appears in the source. Don’t change it.

`msgstr` is where you put your string translation. At beginning it would be empty, you should change it. 

{% highlight python %}
#: views.py:11
msgid "I'm bilingual cat and know how to meow in several languages: Meow."
msgstr "Soy un gato bilingüe y sé como hacer meow en varios idiomas: Meow."

#: views.py:12
msgid "I'm not angry, go to kill yourself."
msgstr "No estoy molesto, mátate."
{% endhighlight %}


After creating our message file, we need to transform it into a more efficient form, a **.mo** file extension. It's a optimized binary file. 

To create those files we need to use the command
`django-admin compilemessages` in our project root directory. 

It would create or update our `.mo` file from a `.po` extension file. 




## Django settings 

We need to indicate to our Django app that we want to use an extra language. By default Django set `LANGUAGE` config var with this value:

{% highlight python %}
LANGUAGES = [
    ('en-us', 'English'),
]
{% endhighlight %}

We are going to add Spanish to django app available languages  `('es-ES', _('Spanish'))`(note, it's `es-ES` not `es_ES`). 

{% highlight python %}
LANGUAGES = [
    ('en-us','English'),
    ('es-ES', 'Spanish')
]
{% endhighlight %}


There are several ways to activate/using Django translations. Manually or by URL Prefix. 

## Activating translation manually

Let's to activate Spanish language in our view, in order to do that, we need to import `translation` utils and activate it using `translation.activate()`. It expects a language code as parameter. You can do that wharever you want, `views.py`, `models.py` or any Django file. 

{% highlight python %}
from django.views.generic import TemplateView
from django.utils.translation import ugettext_lazy as _

from django.utils import translation


class CatQuotesView(TemplateView):
    template_name = "quotes.html"

    def get_context_data(self, *args, **kwargs):
        context = super(CatQuotesView, self).get_context_data(**kwargs)

        SPANISH_LANGUAGE_CODE = 'es-ES'
        translation.activate(SPANISH_LANGUAGE_CODE)

        cat_quote_one = _("I'm bilingual cat and know how to meow in several languages: Meow.")
        cat_quote_two = _("I'm not angry, go to kill yourself.")
        context.update({
            'cat_quote_one': cat_quote_one,
            'cat_quote_two': cat_quote_two
        })
        return context
{% endhighlight %}

Let's visit our cat quotes again `http://localhost:8000/cats/quotes/`.

{% include image.html path="posts/screen_2.png"
                      path-detail="posts/screen_2.png"
                      alt="Sample image" %}
{% include image.html path="posts/screen_3.png"
                      path-detail="posts/screen_3.png"
                      alt="Sample image" %}

So, we got our quotes translated into spanish.

### Using strings translations in templates

{% raw  %}
As well as we mark strings for translation in our views, we can do it in our Django template. Before do that, we need to give our template access to some type of tags in order to do that job, put `{% load i18n %}` toward the top of your template. So, after that we are going to use `{% trans %}` tag. Let's add our page title _Cat quotes for dummys_ to `quotes/templates/quotes.html` and translate it to Spanish. 


```html
{% load i18n %}
...............................
...............................
<div class="col-md-8 col-md-offset-2">
    <div>
        <h3 class="text-center">{% trans "Cat quotes for dummys." %}</h3>
    </div>
    <br/>
    <div class="quote"><i class="fa fa-quote-left fa-4x"></i></div>
    <div class="carousel slide" id="fade-quote-carousel" data-ride="carousel" data-interval="3000">
      <!-- Carousel indicators -->
      <ol class="carousel-indicators">
        <li data-target="#fade-quote-carousel" data-slide-to="0" class="active"></li>
        <li data-target="#fade-quote-carousel" data-slide-to="1"></li>
      </ol>
      <!-- Carousel items -->
      <div class="carousel-inner">
        <div class="item active">
            <img class="profile-circle img-responsive" src="https://s-media-cache-ak0.pinimg.com/564x/12/f6/d1/12f6d18125126757df29e733051697b8.jpg" alt="">
            <blockquote>
                <p>{{cat_quote_one}}</p>
            </blockquote>   
        </div>
        <div class="item">
            <img class="profile-circle img-responsive" src="https://66.media.tumblr.com/avatar_ad6c1cb12b85_128.png" alt="">
            <blockquote>
                <p>{{cat_quote_two}}</p>
            </blockquote>
        </div>
      </div>
    </div>
</div>  
 
```

{% endraw  %}

it should looks like this
{% include image.html path="posts/screen_4.png"
                      path-detail="posts/screen_4.png"
                      alt="Sample image" %}


Our title is already marked for translation. To update our translation file, re-run `django-admin makemessages -l es_ES` command in `quotes` app root dir and add the translation for the new entry in our `.po` file `quotes/locale/es_ES/LC_MESSAGES/django.po`


{% highlight python %}
#: templates/quotes.html:81
msgid "Cat quotes for dummys."
msgstr "Citas de gatos para principiantes."
{% endhighlight %}
Compile again using `django-admin compilemessages` and we should get this.

{% include image.html path="posts/screen_5.png"
                      path-detail="posts/screen_5.png"
                      alt="Sample image" %}


__Note:__ remember that we have activated Spanish as default language in our view.


## Activating translation by URL prefix

As you can notice, it would be so tedious activate and deactivate our language manually. Even more, we want to be able to use certain language given a prefix in our URL. 

Adding the language prefix to the root of the URL patterns to make it possible for `LocaleMiddleware` to detect the language to activate from the requested URL.

To do that, first, we need to add `django.middleware.locale.LocaleMiddleware` in our `MIDDLEWARE` setting. 

{% highlight python %}
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django.middleware.locale.LocaleMiddleware'
]
{% endhighlight %}

That middleware takes care of activating a language from our url lang prefix, i.e `en-us/any_url`.

Also, we should allow language codes in our urls. Using `i18n_patterns()` we can archive that. 

This function can be used in a root URLconf and Django will automatically prepend the current active language code to all URL patterns defined within. 

Our new `/cats/urls.py` should looks like this

{% highlight python %}
from django.conf.urls import url, include
from django.contrib import admin
from django.conf.urls.i18n import i18n_patterns


urlpatterns = [
    url(r'^admin/', admin.site.urls),
]

urlpatterns += i18n_patterns(
    url(r'cats/', include('quotes.urls', namespace='cat_quotes')),
)
{% endhighlight %}
Finally, we need to remove manually language activation from our views. We no longer need it.


{% highlight python %}

from django.views.generic import TemplateView
from django.utils.translation import ugettext_lazy as _


class CatQuotesView(TemplateView):
    template_name = "quotes.html"

    def get_context_data(self, *args, **kwargs):
        context = super(CatQuotesView, self).get_context_data(**kwargs)

        cat_quote_one = _("I'm bilingual cat and know how to meow in several languages: Meow.")
        cat_quote_two = _("I'm not angry, go to kill yourself.")
        context.update({
            'cat_quote_one': cat_quote_one,
            'cat_quote_two': cat_quote_two
        })
        return context

{% endhighlight %}
Let's visit `http://localhost:8000/en-us/cats/quotes/`

{% include image.html path="posts/screen_6.png"
                      path-detail="posts/screen_6.png"
                      alt="Sample image" %}

Now, our spanish version `http://localhost:8000/es-es/cats/quotes/`

{% include image.html path="posts/screen_7.png"
                      path-detail="posts/screen_7.png"
                      alt="Sample image" %}


__Note:__ Language pre ix in our url must match exactly from our defined language codes in the `settings.py` file.

That is, we got our cat quote app translated in two languages. 

If you found this post useful, share and spread the word. 
