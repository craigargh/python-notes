# Django Rest Framework Notes

The Django Rest Frameowork is a plugin for Django designed for building web APIs.

## Setup

The Django Rest Framework requires Django to be installed:

```bash
pip install django djangorestframework
```

To start a new Django project you can use the `django-admin startpoject` command to automatically generate boilerplate Django files:

```bash
django-admin startpoject tutorial
```

Django uses apps that make up the user-defined features of the application. To create a new app and generate all of the Django boilerplate:

```bash
python manage.py startapp snippets
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

The `status` module provides identifiers for status codes, such as `HTTP_400_BAD_REQUEST`.

There are two ways to write views:
1. Function based views with the `@api_view` wrapper
1. Class based views with the `APIView` class

The wrapper and class do multiple things related to requests. They make sure they receive a `Request` object, and add context information to the `Response` so that it knows what format to return the data (e.g. XML, JSON, HTTP).

```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.framework import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


@api_view(['GET', 'POST'])
def snippet_list(request):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Reponse(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

View functions that use `@api_view` can have an optional `format` argument that contains the request format type. To use this argument, you need to add `format_suffix_patterns()` argument to the app's `url.py` file:

```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>', views.snippet_detail),
]

urlpatterns = format_suffic_patterns(urlpatterns)
```

With the `httpie` library you can make requests to the API:

```bash
http http://localhost:8000/snippets/
http http://localhost:8000/snippets/1/
```

You can state the format you want the data to be returned as using the `ACCEPT` flag of `httpie`

```bash
http http://localhost:8000/snippets/ ACCEPT:application/json
http http://localhost:8000/snippets/ ACCEPT:text/html
```

When using the `POST` method you can specify the sent format with a `--json` or `--form` flag:

```bash 
http --json POST http://localhost:8000/snippets/ code="print('hello')"
http --form POST http://localhost:8000/snippets/ code="print('hello')"
```

The `--debug` flag shows additional debug information like the request headers.

When you make a request from a web, the API returns `html` by default. This is based on the client request, just like the above examples. 


## Class-based Views

Class-based views can be created using inheritance. The `rest_framework.views.APIView` class provides a generic view with methods for `get`, `post`, `delete`, and `put`.

Each of these methods should return a `Response` object or an exception.

```python
from rest_framework.views import APIView 
from rest_framework.response import Response
from rest_framework import status
from djano.http import Http404

from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


class SnippetList(APIView):
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_404_NOT_FOUND)


class SnippetDetail(APIView):
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(data=serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)

        if serializer.is_valid():
            serializer.save()
            return Response(data=serializer.data)
        return Response(serializer.errors, status=status.HTTP_404_NOT_FOUND)

    def delete():
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

```

The `.as_view()` method is called to use class-based views in `urls.py`:

```python
urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
]
```

Mixins can provide reusable code for common tasks. For example the `CreateModelMixin` class provides reusable code for creating records. When using mixins, the `GenericAPIView` is used instead of the `APIView` class

```python
from rest_framework import mixins, generics

class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippets.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        self.create(request, *args, **kwargs)
```

The `ListModelMixin` provides the `.list()` method and the `CreateModelMixin` provides the `.create()` method.


```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        self.destroy(request, *args, **kwargs)
```

The `UpdateModelMixin` and `DestroyModelMixin` provide the `update` and `destroy` method respectively.

Another alternative to defining each method's behaviour or using mixins, is to use generic views that have all of the default behaviour pre-written.

The records to use is defined in the `queryset` attribute and the serialiser is set in the `serializer_class` attribute


```python
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```
