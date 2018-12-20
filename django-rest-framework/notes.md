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

There are different fields available for different data types:
- `DateTimeField`
- `CharField`
- `TextField`
- `BooleanField`

All of the fields can be configured with different arguments. Some arguments of note:
- `default`: The default value of the field, if no value is provided when the object is instantiated, this is the value that will be saved to the DB
- `choices`: Limits the values that can be input for a character field. For example if you have a `countries` field you would want to limit it to a list of countries in the world
- `auto_add_now`: Sets a datetime field's value to the current time if it isn't set

To set the default order of objects that will be returned the `ordering` attribute is set in the `Meta` class. In this case it is ordered by the `created` field. To order the objects in descending order a `-` is prepended to the name of the field e.g. `ordering = ('-created')`.

To use the models to create the tables and columns in the database you use the migration commands. 

First you need to make the migrations for the app:

```bash
python manage.py makemigration snippets
```

This will generate a migrations file.

Next you need to apply the migrations to apply the changes to the database:

```bash
python manage.py migrate
```

## Serializers

A serializer class defines how to convert a model into different representations, like `json` or `xml`.

Serializers are created in the `serializers.py` file of an app. There are multuple ways to set up a serializer.

This serializer defines the behaviour when a model is created or updated:

```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choise=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self):
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()

        return instance
```

The `create` and `update` define the behaviour when the `serializer.save()` method is called. The attributes at the start of the class determine the fields of the serializer and their data types.

The `style` argument of the `code` field sets how the field will be displayed on an `html` page.

`ModelSerializers` are a more concise way of creating a serializer from its model definition:

```python
from rest_framework import serializers
from snippets.models import Snippet


class SnippetSerializer(serializers.ModelSerializer):
    model = Snippet
    fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```

The `ModelSerializer` has default implementations for `create` and `update`, which can be overwritten.


## Requests and Responses

The rest framework uses a `Request` object that extends Django's `HttpRequest`. The `request.data` attribute contains the data to Django's `request.POST`, `request.PUT`, and `request.PATCH` methods.

The rest framework `Response` object takes unrendered content and works out the correct type of data to return to the client.


