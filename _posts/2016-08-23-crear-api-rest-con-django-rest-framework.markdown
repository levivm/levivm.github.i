---
title: "Crear un API REST con Django Rest Framework - Parte I"
layout: post
date: 2016-08-23 04:38:13
image: 'https://www.dabapps.com/static/img/technologies/rest.png'
description: Tutorial de como crear un API REST usando Django Rest Framework (DRF).
tag:
- api
- rest
- django
- python
- django rest framework
blog: true
jemoji: ':doughnut:'
author: "Levi Velázquez"
---

# Django Rest Framework (DRF)

DRF es una librería que nos permite construir un API REST sobre Django de forma sencilla. Ofreciendo una alta gama de métodos y funciones para el manejo, definición y control de nuestros recursos(endpoints). Si no sabes cuáles son los pátrones comunes a la hora de diseñar un API REST, puedes leerlo acá en mi [entrada](http://levipy.com/introduccion-al-diseno-de-una-api-rest/) anterior.

Comencemos nuestra instalación.

## Creando nuestro directorio base

Debemos crear el directorio donde va vivir nuestro código

```bash
mkdir tutorial
cd tutorial
```

## Configurando el ambiente 

Antes de comenzar, primero debemos crear nuestro entorno virtual. Esto nos permitirá aislar por completo las dependencias del proyecto, de tal manera, no habrá conflicto con las dependencias y librerías locales existentes en nuestro sistema. Para conocer más a fondo respecto a la instalación de entornos virtuales, puedes revisar mi anterior [tutorial](http://localhost:3000/virtualenv-and-virtualenvwrapper-tutorial/).

```bash
virtualenv tutorial
source tutorial/bin/activate  # On Windows use `env\Scripts\activate`
```
Una vez creado y activado nuestro entorno virtual, procedemos a instalar los paquetes necesarios. 

```bash
pip install django
pip install djangorestframework
```

## Creando nuestro proyecto y la configuración inicial

Una vez instalado DRF en nuestro entorno virtual, procedemos a crear nuestro proyecto en Django y establecer la configuración inicial. Para efectos de este tutorial vamos a crear un API para la obtención de información de películas y series, se llamará _webflix_. 

```bash
django-admin.py startproject webflix
cd webflix
django-admin.py startapp series
```

Ya creada nuestra primera aplicación, procedemos a incorporarla dentro de los módulos instalados de Django al igual que lo haremos con DRF. Para ello, editamos nuestro archivo _/tutorial/settings.py_ y agregamos lo siguiente

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
    'series',
)
```

Ahora comenzaremos a jugar con los serializadores. 

## Serializers y ModelSerializers

Antes de crear nuestro primer serializer, necesitamos crear un modelo el cual vamos a utilizar para trabajar en este tutorial. Crearemos uno llamado `Serie`, dentro de nuestra app _series_, el cual representará toda la información que posee una serie de TV.

```python
from django.db import models


class Serie(models.Model):

    HORROR = 'horror'
    COMEDY = 'comedy'
    ACTION = 'action'
    DRAMA = 'drama'

    CATEGORIES_CHOICES = (
        (HORROR, 'Horror'),
        (COMEDY, 'Comedy'),
        (ACTION, 'Action'),
        (DRAMA, 'Drama'),
    )

    name = models.CharField(max_length=100)
    release_date = models.DateField()
    rating = models.IntegerField(default=0)
    category = models.CharField(max_length=10, choices=CATEGORIES_CHOICES)
```

Luego necesitamos crear las migraciones de Django 

```bash
./manage.py makemigrations series
./manage.py migrate  
```

Una parte vital de Django Rest Framework es la habilidad de poder serializar y deserializar nuestras instancias o recursos en algún tipo de representación más fácil de manejar y transmitir. 

Para aprender un poco más de este proceso, vamos a crear un serializador que nos ayudará a transformar y manejar nuestras instancias de _series_ en formato _JSON_. Para ello, vamos a crear un archivo dentro de nuestra app _series_ llamado `serializers.py`. Dentro de él van a recidir las declaraciones de nuestros serializadores relacionados con nuestra recien creada app.

```python
from rest_framework import serializers
from .models import Serie


class SerieSerializer(serializers.Serializer):
    pk = serializers.IntegerField(read_only=True)
    name = serializers.CharField()
    release_date = serializers.DateField()
    rating = serializers.IntegerField()
    category = serializers.ChoiceField(choices=Serie.CATEGORIES_CHOICES)

    def create(self, validated_data):
        """
        Create and return a new `Serie` instance, given the validated data.
        """
        return Serie.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Serie` instance, given the validated data.
        """
        instance.name = validated_data.get('name', instance.name)
        instance.release_date = validated_data.get('release_date', instance.release_date)
        instance.rating = validated_data.get('rating', instance.rating)
        instance.category = validated_data.get('category', instance.category)
        instance.save()
        return instance
```
Basicamente en la primera parte del serializador definimos los atributos que queremos representar, cada atributo viene acompañado de un tipo, muy al estilo de los _forms_ en Django.

Los serializadores en DRF necesitan dos métodos fundamentales, `create()` y `update()`, los cuales funcionan para crear y actualizar la instancia usando el serializador. 

Ahora es momento de ver cómo inicializar nuestro serializador.

## Trabajando con Serializadores

Para comenzar a probar nuestro serializador de una forma rápida, vamos abrir nuestra consola de Django

```bash
./manage.py shell
```

Ahora, importamos nuestro modelo, el serializador y otras utilidades para crear varias instancias del modelo `Serie`

```python
from series.models import Serie
from series.serializers import SerieSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from datetime import datetime


release_date = datetime.strptime('17-04-2011', '%d-%m-%Y').date()
serie = Serie(name='Game of Thrones', category='drama', release_date=release_date)
serie.save()

release_date = datetime.strptime('24-06-2015', '%d-%m-%Y').date()
serie = Serie(name='Mr. Robot', category='drama', release_date=release_date)
serie.save()

```

Ya creada dos instancias podemos comenzar a jugar con nuestros serializadores. Vamos a serializar una de ellas.

```python
serializer = SerieSerializer(serie)
serializer.data
>{'category': 'drama',
 'name': u'Mr. Robot',
 'pk': 2,
 'rating': 0,
 'release_date': '2015-06-24'}
``` 
En este paso, hemos transformado nuestra instancia en un tipo de dato nativo de Python, en este caso un _diccionario_ (_dict_ para futuras referencias). Para finalizar el proceso de serialización debemos transformar nuestro _dict_ en _JSON_, para ello, usaremos la clase `JSONRenderer` que trae DRF y su método `render()`.

```python
content = JSONRenderer().render(serializer.data)
content
> '{"pk":2,"name":"Mr. Robot","release_date":"2015-06-24","rating":0,"category":"drama"}'
``` 

Si deseamos revertir el proceso, debemos transformar nuestro _JSON_ en un tipo de dato nativo de Python

```python
from django.utils.six import BytesIO

stream = BytesIO(content)
data = JSONParser().parse(stream)
data
> {u'category': u'drama',
 u'name': u'Mr. Robot',
 u'pk': 2,
 u'rating': 0,
 u'release_date': u'2015-06-24'}
``` 

Luego, convertimos nuestro tipo de dato nativo, en este caso un _dict_, a una instancia del modelo _Serie_ con todos sus atributos establecidos.

```python
serializer = SerieSerializer(data=data)
serializer.is_valid()
> True

serializer.validated_data
> OrderedDict([(u'name', u'Mr. Robot'), (u'release_date', datetime.date(2015, 6, 24)), (u'rating', 0), (u'category', 'drama')])

serie = serializer.save()
serie
> <Serie: Serie object>

# Borramos la serie creada para continuar con nuestro ejemplo
serie.delete()
``` 

Acabamos de crear una instancia de nuestro modelo partiendo de un _dict_. Podemos notar una similitud entre trabajar con serializadores y forms de Django. El proceso para crear una instancia es parecido:

1. Inicializamos nuestro serializador con la data requerida
2. Validamos que la data sea correcta
3. Si no existe ningún error, podemos inspeccionar dicha data para manipularla
4. Ejecutamos el metodo `.save()` para crear la instancia. 

No solamente podemos serializar una instancia, nuestro serializador nos permite transformar varias instancias o un queryset. Para ello, sólo necesitamos pasarle el queryset y el flag `many=True` el cual indica que el objeto a serializar es un conjunto de instancias.

```python
serializer = SerieSerializer(Serie.objects.all(), many=True)
serializer.data
>[
  OrderedDict([('pk', 1), ('name', u'Game of Thrones'), ('release_date', '2011-04-17'), ('rating', 0), ('category', u'Drama')]), OrderedDict([('pk', 2), ('name', u'Mr. Robot'), ('release_date', '2015-06-24'), ('rating', 0), ('category', u'Drama')]),
  OrderedDict([('pk', 3), ('name', u'Mr. Robot'), ('release_date', '2015-06-24'), ('rating', 0), ('category', 'drama')]), OrderedDict([('pk', 4), ('name', u'Mr. Robot'), ('release_date', '2015-06-24'), ('rating', 0), ('category', 'drama')])
 ]
```

## Utilizando ModelSerializers

Como podemos darnos cuenta, nuestra clase `SerieSerializer` está replicando gran parte de la información que está contenida en el modelo `Serie`. DRF ofrece la posibilidad de crear un serializador en base a un modelo previamente definido para evitar la duplicidad de información. Al igual que lo hace los forms en Django, definiendo un `ModelForms` en base a un modelo existente. 

Para ello, debemos importar la clase `ModelSerializer` y redefinir nuestro serializador. Abrimos el archivo `series/serializer.py`  y lo modificamos.

```python
class SerieSerializer(serializers.ModelSerializer):
    class Meta:
        model = Serie
        fields = ('id', 'name', 'release_date', 'rating', 'category')
```

Redefinido nuestro serializador, volvemos a consola y procedemos a probarlo. 

```python
from series.serializers import SerieSerializer

serializer = SerieSerializer()
print(repr(serializer))
> SerieSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(max_length=100)
    release_date = DateField()
    rating = IntegerField(required=False)
    category = ChoiceField(choices=(('horror', 'Horror'), ('comedy', 'Comedy'), ('action', 'Action'), ('drama', 'Drama')))
```

Al igual que nuestro serializador anterior, el nuevo posee los mismos atributos. Además, ya contiene implementado los métodos `.create()` y `.update()`, Los mismos son dados por defecto.

## Escribir vistas normales en Django usando un serializador

Luego de definido nuestro serializador, vamos a crear nuestra primera API usando vistas funcionales o vistas simples en Django. Cabe destacar que soy partidario de usar Vistas Basadas en Clases pero por el momento lo haremos de esta forma por fines prácticos. 

Pero antes, crearemos una subclase de `HttpResponse` para poder permitirle a nuestras vistas, devolver el contenido en formato `JSON`. Abrimos el archivo `series/views.py` y agregamos nuestra subclase.

```python
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from series.models import Serie
from series.serializers import SerieSerializer

class JSONResponse(HttpResponse):
    """
    An HttpResponse that renders its content into JSON.
    """
    def __init__(self, data, **kwargs):
        content = JSONRenderer().render(data)
        kwargs['content_type'] = 'application/json'
        super(JSONResponse, self).__init__(content, **kwargs)
```
Ahora, vamos a crear nuestros primeros servicios de nuestra API. Vamos a permitirle al usuario poder listar las series y crear nuevos registros de las mismas. 

```python
@csrf_exempt
def serie_list(request):
    """
    List all code serie, or create a new serie.
    """
    if request.method == 'GET':
        series = Serie.objects.all()
        serializer = SerieSerializer(series, many=True)
        return JSONResponse(serializer.data)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SerieSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JSONResponse(serializer.data, status=201)
        return JSONResponse(serializer.errors, status=400)
```

Debido a que estamos permitiendo a un usuario crear un nuevo registro de una serie, usando el método `POST`, debemos indicarle a Django que está vista no debe solicitar el `token csrf` que usualmente se maneja, debido a que nuestro servicio será público y el usuario no necesariamente posee ese token, esto para nuestros fines prácticos. Para ello, usamos el decorador `@csrf_exempt`. 

Usualmente, esto no es lo que se debería hacer, DRF ofrece mecanismos para manejar de una mejor forma estos casos. 

Ahora, vamos a permitirle al usuario obtener, actualizar o borrar una serie

```python
@csrf_exempt
def serie_detail(request, pk):
    """
    Retrieve, update or delete a serie.
    """
    try:
        serie = Serie.objects.get(pk=pk)
    except Serie.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SerieSerializer(serie)
        return JSONResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SerieSerializer(serie, data=data)
        if serializer.is_valid():
            serializer.save()
            return JSONResponse(serializer.data)
        return JSONResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        serie.delete()
        return HttpResponse(status=204)
```

Finalmente, agregamos nuestras urls para exponer los servicios. Creamos un archivo `serie/urls.py` dentro de nuestra app `series`.

```python
from django.conf.urls import url
from series import views


urlpatterns = [
    url(r'^series/$', views.serie_list),
    url(r'^series/(?P<pk>[0-9]+)/$', views.serie_detail),
]
```


Procedemos a abrir nuestro archivo de urls principal `webflix/urls.py` para importar las urls establecidas en nuestra app series. 

```python
....
from django.conf.urls import url, include
...

urlpatterns = [
  ....
    url(r'^', include('series.urls')),
    ....
]
```
Cabe destacar que debido a la practicidad del ejemplo, hay casos, los cuales no se están tomando en cuenta. Por ejemplo, qué ocurre si el usuario envía una petición mal formada, que puede originar un error 500. Para ello, existen varias formas de manejar ciertas excepciones. Las mismas no están incluidas en el alcance de este tutorial.


## Probando nuestra API

Primero, debemos levantar nuestro servidor

```
./manage.py runserver

Performing system checks...

System check identified no issues (0 silenced).
August 03, 2016 - 08:23:45
Django version 1.9.7, using settings 'webflix.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Ahora, abrimos otra terminal para probar nuestra API

Podemos solicitar los servicios API usando [curl](https://curl.haxx.se/) or [httpie](https://github.com/jkbrzt/httpie#installation). Httpie es un servidor http ligero, muy fácil de usar. 

Instalamos httpie

```
pip install httpie
```
Ahora, vamos a listar todas las series

```bash
$http http://127.0.0.1:8000/series/

HTTP/1.0 200 OK
Content-Type: application/json
Date: Wed, 03 Aug 2016 08:29:53 GMT
Server: WSGIServer/0.1 Python/2.7.10
X-Frame-Options: SAMEORIGIN

[
    {
        "category": "drama",
        "id": 1,
        "name": "Game of Thrones",
        "rating": 0,
        "release_date": "2011-04-17"
    },
    {
        "category": "drama",
        "id": 2,
        "name": "Mr. Robot",
        "rating": 0,
        "release_date": "2015-06-24"
    }
]
```
O podemos solicitar una serie por su `id`

```bash
$http http://127.0.0.1:8000/series/1/

HTTP/1.0 200 OK
Content-Type: application/json
Date: Wed, 03 Aug 2016 08:29:53 GMT
Server: WSGIServer/0.1 Python/2.7.10
X-Frame-Options: SAMEORIGIN

{
    "category": "Drama",
    "id": 1,
    "name": "Game of Thrones",
    "rating": 0,
    "release_date": "2011-04-17"
}
```

Para crear una serie, hacemos un `POST` a `/series/`

```bash
$http -j POST http://127.0.0.1:8000/series/ category=comedy name=Lost release_date=2004-09-22

HTTP/1.0 201 Created
Content-Type: application/json
Date: Wed, 03 Aug 2016 08:30:50 GMT
Server: WSGIServer/0.1 Python/2.7.10
X-Frame-Options: SAMEORIGIN

{
    "category": "drama",
    "id": 4,
    "name": "Lost",
    "rating": 0,
    "release_date": "2004-09-22"
}
```

Obviamente se estarán preguntando, qué le pasa, por qué la categoría de LOST es comedia, pero tranquilos, _I know_, es drama. Para enmendar mi error, vamos actualizar la información de nuestra serie haciendo un `PUT` a la url `/series/4/` con la nueva categoría.

```bash
$http -j PUT http://127.0.0.1:8000/series/4/ category=drama name=Lost release_date=2004-09-22  

HTTP/1.0 200 OK
Content-Type: application/json
Date: Wed, 03 Aug 2016 08:32:53 GMT
Server: WSGIServer/0.1 Python/2.7.10
X-Frame-Options: SAMEORIGIN

{
    "category": "drama",
    "id": 4,
    "name": "Lost",
    "rating": 0,
    "release_date": "2004-09-22"
}
```
  

DRF también ofrece por defecto una interfaz web el cual permite interactuar con el API. Para ello, abre un navegador y visita la misma url antes solicitada. 


## Siguiente paso

Por el momento, sabemos como crear serializadores y que los mismos se comportan de manera muy similar a los forms de Django. Además, logramos crear un API REST funcional que por el momento sólo sirve data en formato JSON. Es importante aclarar que no se están teniendo en cuenta ciertos factores que deben manejarse a la hora de querer implementar servicios robustos. 

En el siguiente tutorial hablaremos un poco más a fondo de las peticiones y las respuestas. Además, veremos ciertos métodos para mejorar lo anteriormente logrado. Como por ejemplo, utilizar Vistas Basadas en Clases con DRF.

Para cerrar, no puede faltar el gif chevere.

{: .text-center}
When the boss finally decides to drop the IE compatibility
![Markdowm Image](http://ljdchost.com/ce9hqIt.gif)

Si les pareció útil el post, comparte :)

P.D: Si no han usado [Vistas Basadas en Clases](https://docs.djangoproject.com/en/1.10/topics/class-based-views/), _you should_. Si las han usado, entonces [acá](https://ccbv.co.uk/) podrán encontrar cada vista generica con su descripción de atributos y métodos para tratar de DRY.