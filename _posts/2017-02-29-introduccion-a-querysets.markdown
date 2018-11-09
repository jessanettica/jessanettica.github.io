---
layout: post
title:  "Introducción a los Querysets de Django"
date:   2017-02-29 07:45:27 -0800
tags: fitness
image_sliders:
  - slider4
style: |
  .post-title {
    font-family: 'Playfair Display', serif;
    font-size: 45px;
  }
  .post-content p {
    font-family: 'Pontano Sans', sans-serif;
    font-size: 20px;
  }
  pre, code {
    background-color: #ffd7d7;
  }
photos: ["/img/clouds.jpg"]
---

{% include slider.html selector="slider4" %}

*Que es un Queryset? Es una lista?*

Eh... no pienses eso porque te vas a confundir después. Parece una lista pero no lo es.

{% highlight shell %}
>>> p = BlogPost.objects.all()
>>> type(p)
>>> <class 'django.db.models.query.QuerySet'>
{% endhighlight %}

Lo ves? No es una lista. Piensa en un Queryset como una colección de objetos de un modelo determinado. Es básicamente una representación de una o varias filas de una base de datos que puede ser filtrada or ordenada con un comando.


&nbsp;

*Como puedo jugar con comandos de Django para mi projecto?*

En el directorio que contiene tu proyecto escribe este comando:

{% highlight shell %}
>>>python manage.py shell
{% endhighlight %}

Depende de las modelos que tengas, pero por ejemplo si tienes una modelo llamada Articulo puedes hacer esto:

{% highlight shell %}
>>>Articulo.objects.all()

Traceback (most recent call last):
      File "<console>", line 1, in <module>
NameError: name 'Articulo' is not defined
{% endhighlight %}

Ups. Se nos olvido importar la modelo.

{% highlight shell %}
>>>from blog.models import  Articulo
{% endhighlight %}

Ahora trata otra vez y funcionara :)

&nbsp;

*Como puedo crear un objeto?*

{% highlight python %}
# esto va a obtener la instancia del modelo Usuario que vamos a guardar como el escritor del objeto del modelo Artículo que vamos a crear.
yo = Usuario.objects.get(username='jessanettica')
# ahora creamos el objeto
Articulo.objects.create(autor=yo, titulo="Titulo de Ejemplo", text="Holaaaa estas ahi?")
Articulo.objects.all()
{% endhighlight %}

&nbsp;

*Que tal si solo quiero conseguir Artículos escritos por un usuario en particular?*

Primero tienes que obtener el usuario que quieres (usemos el que ya tenemos de ejemplo) y haz algo como esto:
{% highlight shell %}
>>> Articulo.objects.filter(autor=yo)
[<Articulo: Titulo de Ejemplo>, <Articulo: Ejemplo 2>, <Articulo: Mi Tercer Articulo!>, <Post: Cuarto Ejemplo>]
{% endhighlight %}
Bien hecho.

&nbsp;

*Puedes encadenar varios filtros?*

{% highlight python %}
# Regresa todos los artículos escritos después de 2014 excepto los que escribió
# Jessica Lopez. Ordenalos por nombre del autor, luego cronológicamente, con los mas
# recientes primero.
Articulo.objects.filter(ano_escrito__gt=2014) \
            .exclude(autor='Jessica Lopez') \
            .order_by(‘autor’, '-ano_publicado')
{% endhighlight %}

&nbsp;

*My amiga dijo que Querysets son geniales porque son flojos. Que quiere decir con "flojos"?*

Que se pasan todo el dia en el sillon jaja :P Lo que quiere decir es que cuando creas un Queryset, no está necesariamente usando la base de datos. Puedes encadenar varios filtros y Django no va a utilizar la base de datos hasta que el Queryset sea evaluado (hablaremos más de qué significa ser "evaluado" en otra sección). Por ahora mira esto:

{% highlight shell %}
>>> a = Articulo.objects.filter(titulo__startswith="Como Hacer")
>>> a = a.filter(fecha_de_pub__lte=datetime.date.today())
>>> a = a.exclude(texto__icontains="arroz")
>>> print(z)
{% endhighlight %}

Cuántas veces crees que Django utilizó la base de datos? Se ve como si la use tres veces, no? Django la utilizo una sola vez, en la última línea cuando se imprime el queryset. Esto es genial porque hace que las cosas funcionen más rápido sin tener que utilizar la base de datos varias veces.

&nbsp;

*Entonces cuando son evaluados los Querysets si no lo son cuando son filtrados?*

Un Queryset puede ser creado, filtrado, cortado, o pasado de aquí allá sin utilizar la base de datos, pero para utilizar esto a tu ventaja, tienes que saber cuando es evaluado.

#### Iteración

Un Queryset es iterable. Utiliza la base de datos la primera vez que iteran sobre el.

{% highlight shell %}
>>>for articulo in Articulo.objects.all():
...    print articulo.titulo
{% endhighlight %}

#### Cortando (más o menos)

Un Queryset puede ser cortado como una lista utilizando los métodos que vienen incluidos con Python (pero recuerda que un Queryset no es una lista!) Cortando un Queryset que no ha sido evaluado usualmente regresa otro Queryset que no ha sido evaluado, pero Django utilizará la base de datos si tu utilizas el parámetro del "paso" y regresara una lista. Otra cosa que no es tan genial es que cuando cortas un Queryset que no ha sido evaluado y regresa otro Queryset no evaluado ya no puedes modificar más ese Queryset (filtrar más, cambiar el orden, et cetera).

#### repr()

Un Queryset es evaluado cuando llamas el método repr(). Este método existe para tu beneficio, para que puedas ver tus resultados inmediatamente en la consola interactiva de Django.

#### len()

Un Queryset es evaluado cuando llamas len(). Probablemente adivinaste, pero regresa qué larga es tu lista de resultados. Si estás utilizando len() es porque quieres saber el número de objetos en tu resultado - utiliza `count()` en vez de `len()`. Por ejemplo:

{% highlight shell %}
>>>Articulo.objects.filter(titulo__startswith="Como hacer").count()
{% endhighlight %}
Esto te dara el número de Artículos que tienen un título que empieza con "Como hacer"

#### list()

Puedes forzar la evaluación de un Queryset llamando `list()`. La mayoría del tiempo la verdad no necesitas hacer esto.
{% highlight shell %}
>>>lista_de_articulos = list(Articulo.objects.all())
{% endhighlight %}

#### El condicional "If" o `bool()`

Utilizar un Queryset en un contexto booleano causará que el queryset sea evaluado. Si hay por lo menos un resultado, regresará True, y si no False

&nbsp;

*Ok ahora se cuando son evaluados pero no se que significa ser "evaluado".*

Tienes razón! Nunca lo explique. Cuando un Queryset es evaluado significa que todas las líneas que coinciden con lo que pediste en tu comando son tomadas de la base de datos y convertidas en modelos de Django.


&nbsp;

*Se guardan en memoria los Querysets?*

Si! Bueno, más o menos. Son "cached" o guardados en memoria en algunas situaciones.
{% highlight python %}
articulos = Articulo.objects.filter(texto__icontains="comida")  # ovio tengo hambre jaja
# En la próxima línea el comando es ejecutado y guardado / "cached"
for articulo in articulos:
    print(articulo.titulo)
# La memoria es utilizada la próxima vez que iteran sobre el Queryset y no hay necesidad de utilizar la base de datos.
for articulo in articulos:
    print(articulo.texto)
{% endhighlight %}

Lo mismo va cualquier vez que un Queryset es evaluado (puedes utilizar la lista que hicimos antes). Por ejemplo digamos que utilizamos un "if" antes de iterar con "for":

{% highlight python %}
articulos = Articulo.objects.filter(texto__icontains="comida")
# El condicional `if` causa que el queryset sea evaluado
if articulos:
    # Se utiliza lo que está guardado en memoria para la próxima iteración
    for articulo in articulos:
        print(articulo.titulo)
{% endhighlight %}

Pero qué tal si no quieres todos los artículos? Solo querias saber si había al menos uno, verdad? En ese caso, utilizar `exists()` es mejor.

{% highlight python %}
articulos = Articulo.objects.filter(texto__icontains="postre")
# `exists()` previene que todo lo que está en el queryset sea guardado en memoria
if articulos.exists():
    # Ningunas filas fueron obtenidas de la base de datos. Eso significa que ahorramos bandwidth y memoria.
    print("Ah un artículo sobre postres!")
{% endhighlight %}

&nbsp;

Bueno eso es todo por ahora, pero espero pronto escribir algo sobre como optimizar comandos para Django. 👋
