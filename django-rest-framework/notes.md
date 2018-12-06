# Django Rest Framework Notes

The Django Rest Frameowork is a plugin for Django designed for building web APIs.

## Setup

The Django Rest Framework requires Django to be installed:

```bash
pip install django djangorestframework
```

To start a new Django project you can use the `django-admin startpoject` command to automatically generate boilerplate Django files:

```bash
django-admin startpoject guitarpractice
```

Django uses apps that make up the user-defined features of the application. To create a new app and generate all of the Django boilerplate:

```bash
python manage.py startapp exercises
```

After creating a new app you need to add it to the Django `settings.py` file to include it in the application. When using the Django Rest Framework this also needs to be added to the `settings.py` file:

```bash
INSTALLED_APPS = [
    'rest_framework',
    'snippets.apps.SnippetsConfig',
]
```

## Models

Django uses models to define the table schemas used in the database. This has many benefits. In particular the models can be used as an ORM to create object/instances that are used in Python code that can directly interact with the DB models. It also creates a single source of truth for the database schemas making migrations to the database easier.

Models are defined in apps. 

This model creates a code snippet:


```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())



class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)
```