## martor [![pypi version][1]][2] [![paypal donation][3]][4]

[![license][5]][6] [![python version][7]][8] [![django version][9]][10] [![build][11]][12]

**Martor** is a Markdown Editor plugin for Django, supported for _Bootstrap_ & _Semantic-UI_.


### Features

* Live Preview
* Integrated with [_Ace Editor_](https://ace.c9.io)
* Supported with [_Bootstrap_](https://getbootstrap.com) and [_Semantic-UI_](https://semantic-ui.com)
* Supported Multiple Fields [_fixed this issue_](https://github.com/agusmakmun/django-markdown-editor/issues/3)
* Upload Images to imgur.com _(via API)_ and [custom uploader][13]
* Direct Mention users `@[username]` - _(requires user to logged in)_.
* Supports embed/iframe video from (Youtube, Vimeo, Dailymotion, Yahoo, Veoh, & Metacafe)
* Spellchecking (only supports US English at this time)
* Emoji `:emoji_name:` + Cheat sheets
* Martor Commands Reference
* Supports Django Admin
* Toolbar Buttons
* Highlight `pre`


### Preview

![editor](https://raw.githubusercontent.com/agusmakmun/django-markdown-editor/master/.etc/images/bootstrap/martor-editor.png)

![preview](https://raw.githubusercontent.com/agusmakmun/django-markdown-editor/master/.etc/images/bootstrap/martor-preview.png)


### Requirements

* `Django>=2.0`
* `Markdown>=3.0`
* `requests>=2.12.4`


### Installation

Martor is available directly from [PyPI][2]:

**1.** Installing the package.

```
$ pip install martor
```


**2.** Don't forget to add `'martor'` to your `'INSTALLED_APPS'` setting _(without migrations)_.

```
# settings.py
INSTALLED_APPS = [
    ....
    'martor',
]
```


**3.** Add url pattern to your `urls.py.`

```
# urls.py
# django >= 2.0
urlpatterns = [
    ...
    path('martor/', include('martor.urls')),
]

# django <= 1.9
urlpatterns = [
    ...
    url(r'^martor/', include('martor.urls')),
]
```


**4.** Collect martor's static files in your `STATIC_ROOT` folder.

```
./manage.py collectstatic
```


### Setting Configurations `settings.py`

Please register your application at https://api.imgur.com/oauth2/addclient
to get `IMGUR_CLIENT_ID` and `IMGUR_API_KEY`.

```
# Choices are: "semantic", "bootstrap"
MARTOR_THEME = 'bootstrap'

# Global martor settings
# Input: string boolean, `true/false`
MARTOR_ENABLE_CONFIGS = {
    'emoji': 'true',        # to enable/disable emoji icons.
    'imgur': 'true',        # to enable/disable imgur/custom uploader.
    'mention': 'false',     # to enable/disable mention
    'jquery': 'true',       # to include/revoke jquery (require for admin default django)
    'living': 'false',      # to enable/disable live updates in preview
    'spellcheck': 'false',  # to enable/disable spellcheck in form textareas
    'hljs': 'true',         # to enable/disable hljs highlighting in preview
}

# To show the toolbar buttons
MARTOR_TOOLBAR_BUTTONS = [
    'bold', 'italic', 'horizontal', 'heading', 'pre-code',
    'blockquote', 'unordered-list', 'ordered-list',
    'link', 'image-link', 'image-upload', 'emoji',
    'direct-mention', 'toggle-maximize', 'help'
]

# To setup the martor editor with title label or not (default is False)
MARTOR_ENABLE_LABEL = False

# Imgur API Keys
MARTOR_IMGUR_CLIENT_ID = 'your-client-id'
MARTOR_IMGUR_API_KEY   = 'your-api-key'

# Markdownify
MARTOR_MARKDOWNIFY_FUNCTION = 'martor.utils.markdownify' # default
MARTOR_MARKDOWNIFY_URL = '/martor/markdownify/' # default

# Markdown extensions (default)
MARTOR_MARKDOWN_EXTENSIONS = [
    'markdown.extensions.extra',
    'markdown.extensions.nl2br',
    'markdown.extensions.smarty',
    'markdown.extensions.fenced_code',

    # Custom markdown extensions.
    'martor.extensions.urlize',
    'martor.extensions.del_ins',      # ~~strikethrough~~ and ++underscores++
    'martor.extensions.mention',      # to parse markdown mention
    'martor.extensions.emoji',        # to parse markdown emoji
    'martor.extensions.mdx_video',    # to parse embed/iframe video
    'martor.extensions.escape_html',  # to handle the XSS vulnerabilities
]

# Markdown Extensions Configs
MARTOR_MARKDOWN_EXTENSION_CONFIGS = {}

# Markdown urls
MARTOR_UPLOAD_URL = '/martor/uploader/' # default
MARTOR_SEARCH_USERS_URL = '/martor/search-user/' # default

# Markdown Extensions
# MARTOR_MARKDOWN_BASE_EMOJI_URL = 'https://www.webfx.com/tools/emoji-cheat-sheet/graphics/emojis/'     # from webfx
MARTOR_MARKDOWN_BASE_EMOJI_URL = 'https://github.githubassets.com/images/icons/emoji/'                  # default from github
MARTOR_MARKDOWN_BASE_MENTION_URL = 'https://python.web.id/author/'                                      # please change this to your domain

# If you need to use your own themed "bootstrap" or "semantic ui" dependency
# replace the values with the file in your static files dir
MARTOR_ALTERNATIVE_JS_FILE_THEME = "semantic-themed/semantic.min.js"   # default None
MARTOR_ALTERNATIVE_CSS_FILE_THEME = "semantic-themed/semantic.min.css" # default None
MARTOR_ALTERNATIVE_JQUERY_JS_FILE = "jquery/dist/jquery.min.js"        # default None
```

Check this setting is not set else csrf will not be sent over ajax calls:

```
CSRF_COOKIE_HTTPONLY = False
```


### Usage


#### Model

```
from django.db import models
from martor.models import MartorField

class Post(models.Model):
    description = MartorField()
```


#### Form

```
from django import forms
from martor.fields import MartorFormField

class PostForm(forms.Form):
    description = MartorFormField()
```


#### Admin

```
from django.db import models
from django.contrib import admin

from martor.widgets import AdminMartorWidget

from yourapp.models import YourModel

class YourModelAdmin(admin.ModelAdmin):
    formfield_overrides = {
        models.TextField: {'widget': AdminMartorWidget},
    }

admin.site.register(YourModel, YourModelAdmin)
```


#### Template Renderer

Simply safely parse markdown content as html ouput by loading templatetags from `martor/templatetags/martortags.py`.

```
{% load martortags %}
{{ field_name|safe_markdown }}

# example
{{ post.description|safe_markdown }}
```


Don't miss to include the required css & js files before use.
You can take a look at this folder [martor_demo/app/templates][14] for more details.
The below example is a one of the way to implement it when you choose the `MARTOR_THEME = 'bootstrap'`:

```
{% extends "bootstrap/base.html" %}
{% load static %}
{% load martortags %}

{% block css %}
  <link href="{% static 'plugins/css/ace.min.css' %}" type="text/css" media="all" rel="stylesheet" />
  <link href="{% static 'martor/css/martor.bootstrap.min.css' %}" type="text/css" media="all" rel="stylesheet" />
{% endblock %}

{% block content %}
  <div class="martor-preview">
    <h1>Title: {{ post.title }}</h1>
    <p><b>Description:</b></p>
    <hr />
    {{ post.description|safe_markdown }}
  </div>
{% endblock %}

{% block js %}
  <script type="text/javascript" src="{% static 'plugins/js/highlight.min.js' %}"></script>
  <script>
    $('.martor-preview pre').each(function(i, block){
        hljs.highlightBlock(block);
    });
  </script>
{% endblock %}
```


#### Template Editor Form

Different with *Template Renderer*, the *Template Editor Form* have more css & javascript dependencies.

```
{% extends "bootstrap/base.html" %}
{% load static %}

{% block css %}
  <link href="{% static 'plugins/css/ace.min.css' %}" type="text/css" media="all" rel="stylesheet" />
  <link href="{% static 'plugins/css/resizable.min.css' %}" type="text/css" media="all" rel="stylesheet" />
  <link href="{% static 'martor/css/martor.bootstrap.min.css' %}" type="text/css" media="all" rel="stylesheet" />
{% endblock %}

{% block content %}
  <form class="form" method="post">{% csrf_token %}
    <div class="form-group">
      {{ form.title }}
    </div>
    <div class="form-group">
      {{ form.description }}
    </div>
    <div class="form-group">
      <button class="btn btn-success">
        <i class="save icon"></i> Save Post
      </button>
    </div>
  </form>
{% endblock %}

{% block js %}
  <script type="text/javascript" src="{% static 'plugins/js/ace.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/mode-markdown.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/ext-language_tools.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/theme-github.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/typo.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/spellcheck.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/highlight.min.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/resizable.min.js' %}"></script>
  <script type="text/javascript" src="{% static 'plugins/js/emojis.min.js' %}"></script>
  <script type="text/javascript" src="{% static 'martor/js/martor.bootstrap.min.js' %}"></script>
{% endblock %}
```


### Custom Uploader

If you want to save the images uploaded to your storage,
**Martor** also provides a way to handle this. Please checkout this [WIKI][13]


### Test Martor from this Repository

Assuming you are already setup with a virtual enviroment (virtualenv):

```
$ git clone https://github.com/agusmakmun/django-markdown-editor.git
$ cd django-markdown-editor/ && python setup.py install
$ cd martor_demo/
$ python manage.py makemigrations && python manage.py migrate
$ python manage.py runserver
```


Checkout at http://127.0.0.1:8000/simple-form/ on your browser.


### Martor Commands Reference

![command refference](https://raw.githubusercontent.com/agusmakmun/django-markdown-editor/master/.etc/images/bootstrap/martor-guide.png)


### Notes

**Martor** was inspired by these great projects: [django-markdownx][15], [Python Markdown][16] and [Online reStructuredText editor][17].


[1]: https://img.shields.io/pypi/v/martor.svg
[2]: https://pypi.python.org/pypi/martor

[3]: https://img.shields.io/badge/donate-paypal-blue
[4]: https://www.paypal.com/paypalme/summonagus

[5]: https://img.shields.io/badge/license-GNUGPLv3-blue.svg
[6]: https://raw.githubusercontent.com/agusmakmun/django-markdown-editor/master/LICENSE

[7]: https://img.shields.io/pypi/pyversions/martor.svg
[8]: https://pypi.python.org/pypi/martor

[9]: https://img.shields.io/badge/Django-1.8%20%3E=%203.2-green.svg
[10]: https://www.djangoproject.com

[11]: https://travis-ci.org/agusmakmun/django-markdown-editor.svg?branch=master
[12]: https://travis-ci.org/agusmakmun/django-markdown-editor

[13]: https://github.com/agusmakmun/django-markdown-editor/wiki
[14]: https://github.com/agusmakmun/django-markdown-editor/tree/master/martor_demo/app/templates
[15]: https://github.com/adi-/django-markdownx
[16]: https://github.com/waylan/Python-Markdown
[17]: http://rst.ninjs.org
