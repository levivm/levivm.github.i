---
title: "Patrones y Buenas Prácticas para diseño de un API REST"
layout: post
date: 2016-08-04 04:38:13
image: 'http://nvie.com/img/merge-without-ff@2x.png'
description: Qué es y cómo diseñar un API REST siguiendo los principios básicos estándares
tag: API REST
tags:
- API
- REST
- Patterns
blog: true
jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'

---

### Qué es un API REST

Primero, debemos saber el significado de API, por sus siglas en inglés _Application Programming Interface_. Básicamente, representan una colección de métodos y funciones implementadas en un sistema con el fin de que las mismas sean utilizadas por otros. 

A medida que surgieron APIs sobre aplicaciones y plataformas web, se fue formando el término API REST, el cual se usa para denominar a las API que están implementadas bajo el protocolo HTTP. En otras palabras, un API REST  permite el uso de un servicio(function o método) perteneciente a una plataforma a un usuario externo para que el mismo lo use en una aplicación propia. 

### Características

#### URI's para identificación de recursos

   Un recurso básicamente representa una sección, archivo o contenido que deseamos obtener o modificar de una plataforma web a través de su API REST. Los mismos serán identificados por una URI, la cuál nos ayudará a obtener/modificar el mismo. 
   
   Las URI's deben cumplir con ciertas características: 
   
   - La URI no debe llevar un verbo que implique una acción 
        __Incorrecto__: _/recursos/id/editar_     
        __Correcto__: _/recursos/id/_ 

   - La URI deben identificar solo a un recurso, deben ser únicas.

   - No se debe tomar en cuenta el formato en la construcción de la URI  
      __Incorrecto__: _/recursos/archivo.pdf_   
      __Correcto__: _/recursos/archivo_

   - Debe mantener una jerarquía lógica.  
      __Incorrecto__: _/apartamentos/2/edificio/3_    
      __Correcto__: _/edificios/3/apartamentos/2_


   - Los filtrados de información deben hacerse mediante los parámetros HTTP  
      __Incorrecto__: _/edificios/3/apartamentos/2/orden/asc_      
      __Correcto__: _/edificios/3/apartamentos/2/?orden=asc_  

      
#### Semántica de los métodos HTTP (GET, POST, PUT, PATCH, DELETE)

Una vez conocidas las principales reglas a la hora de construir nuestras URIs, pasamos al siguiente paso, darle un significado semántico o de acción a los métodos HTTP. Ampliando nuestro camino un poco más allá de los dos métodos más utilizados comúnmente: __GET__ y __POST__. 

Los métodos serían los siguientes:

- __GET__: Para consultar y leer recursos
- __POST__: Para crear recursos
- __PUT__: Para editar recursos
- __DELETE__: Para eliminar recursos.
- __PATCH__: Para editar partes concretas de un recurso.

#### Resultado final

Uniendo los dos puntos anteriormente explicados, tendremos como resultado las principales reglas que debemos tomar en cuenta a la hora de diseñar las URL's para nuestra API REST.



__GET /series__ Nos permite acceder al listado de series

__POST /series__ Nos permite crear una nueva serie

__GET /series/123__ Nos permite acceder al detalle de una serie

__PUT /series/123__ Nos permite editar una serie, sustituyendo la totalidad de la información anterior por la nueva.

__DELETE /series/123__ Nos permite eliminar una serie

__PATCH /serie/123__ Nos permite modificar cierta información de una serie, como el nombre

Con esto hemos establecido los principios fundamentales para construir un API REST siguiendo los estándares establecidos.  

En el próximo tutorial, comenzaremos a hacer nuestra primera API REST usando Django y Django Rest Framework.