---
layout: post
title:  "Extensión del Modelo de Usuario de Django"
date:   2017-01-29 10:35:20 -0800
categories: python
image_sliders:
  - slider2
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
photos: ["img/berries.jpg", "img/candy.jpg"]
---

{% include slider.html selector="slider2" %}

El sistema de autenticación de usuarios que viene incluido con Django es genial. Maneja la autenticación y la autorización, pero para muchos de tus proyectos, querrás extenderla y personalizarla. Una de las principales razones por las que querrás personalizar el sistema integrado es almacenar más datos relacionados con el Usuario. Por ejemplo, es posible que desees almacenar la ciudad donde vive el usuario o su cuenta de Instagram. Hay diferentes maneras de extender el modelo de Usuario, y no existe una opción que sea "la mejor" o "la peor". La mejor opción dependerá en tu proyecto y que tanto hayas implementado en tu proyecto cuando decidas extender el modelo de Usuario. Ok, empecemos.

Si estás a medio camino en tu proyecto opcion  # 1 o opcion # 2 son las mejores opciones. Ambas extienden el modelo de usuario incluido sin sustituirlo por tu propio modelo.

### 1. Un enlace uno a uno con un modelo de Usuario (“UserProfile” o Perfil de Usuario)

La mejor opción si no estás al inicio de tu proyecto y necesitas almacenar información adicional sobre el usuario que no está relacionada con el proceso de autenticación. Por lo general, las personas que hacen esto llaman a la tabla UserProfile, pero tu puedes hacer y llamarlo lo que quieras!

{% highlight python %}
from django.contrib.auth.models import User

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    ciudad = models.CharField(max_length=100, blank=True)
    instagram_nombre = models.CharField(max_length=30, blank=True)
{% endhighlight %}


Ahora en el en la consola interactiva de Django puedes hacer:

{% highlight shell %}
>>> u = User.objects.get (username = 'jessica')
>>> instagram_de_jessica = u.userprofile.instagram_nombre
{% endhighlight %}

Como puedes ver, no es un modelo especial de ninguna manera. Es sólo un modelode  Django normal que tiene una relación de uno a uno con el modelo de usuario a través de un OnetoOneField y tendrá su propia tabla en la base de datos. Debido a que UserProfile es sólo otro modelo, no se actualizará automáticamente cuando se crea un nuevo usuario. Para hacer esto automáticamente necesitarás usar las señales de Django para escuchar el evento `post_save` del usuario y actualizar la instancia UserProfile cuando esto suceda (los documentos de Django son [mágicos y hermosos][django-docs])

Quiero mencionar que el código anterior es para Dango 1.10. Si eres como yo y has estado usando Django 1.8 o antes, on_delete ahora puede ser utilizado como el segundo argumento posicional (anteriormente, normalmente sólo se pasaba como un argumento de palabra clave). No es necesario, pero lo será en Django 2.0 así que vale la pena comenzar a usarlo. Hay varias opciones que puedes usar, pero aqui utilize CASCADE, que es como decirle a Django: "Oye, cuando el objeto de referencia se elimine, también elimina los objetos que tienen referencias a el, ¿ok? Gracias."

Algo que vale la pena tener en cuenta es que el uso de un modelo relacionado resulta en consultas (se dice asi? Perdon es que casi nunca hablo de programación en español :))  adicionales a la base de datos o uniones para recuperar los datos. Puedes optimizar sus comandos, pero la mayoría de las veces intentes acceder a datos relacionados, Django realizará una consulta adicional. Esta es la razón por la cual si estás en el principio de un proyecto, un modelo de usuario personalizado puede ser una mejor opción (ve opción # 3).

No olvides que si deseas agregar los campos de tu nuevo UserProfile modelo a la página de usuario en el administrador de Django necesitas registrarlo! En `admin.py` agregalo a la clase UserAdmin que está registrada con la clase User.


### 2. Uso de un modelo proxy

Esta opción es para ti si no necesitas almacenar información adicional en la base de datos, pero quieres agregar métodos adicionales. Ésta opción hereda del modelo pero no crea una nueva tabla en la base de datos; no afecta el esquema de base de datos existente.

{% highlight python %}
from django.contrib.auth.models import User
from .managers import BloggerManager

class Subscriber(User):
    objects = SubscriberManager()

    class Meta:
        proxy = True
        ordering = ('nombre', )

    def plan_de_subscripcion(self):
        ...
{% endhighlight %}

Esto cambia el orden predeterminado por `nombre` y le da un “custom Manager” a Subscriber. `plan_de_subscripcion` es sólo un ejemplo de un método. Puedes crear cualquier método que necesites para tu proyecto.

Después de hacer esto, `User.objects.all()` y `Subscriber.objects.all()` consultará la misma tabla en la base de datos. Genial, no? La única diferencia está en la lógica definida para el proxy.

Si estás al principio de tu proyecto, las dos opciones siguientes son las mejores. No son tan comunes porque cambian significativamente el esquema de la base de datos y también porque la gente normalmente no sabe cómo va a querer personalizar el sistema incluido con Django al principio de su proyecto. Así que si tienes una epifanía de como quieres cambiar la autenticación de usuario en medio de tu proyecto, es mejor escoger option # 1 o opcion # 2.


### 3. Creación de un modelo de usuario personalizado que extiende a AbstractUser

Esta es la mejor opción si te gusta la forma en la que Django maneja la autenticación, pero quieres agregar alguna información adicional directamente en el modelo de usuario, sin tener que crear una extra clase (como en # 2). En un mundo ideal donde se sabe qué información adicional necesitará su modelo de usuario al comienzo de un proyecto, esta es la mejor opción. Es simple y evita la complejidad de configurar # 4, y evita las acrobacias mentales que tendrás que hacer para recordar cómo funciona el sistema de tu aplicación más adelante si haces # 1 o # 2.

Su nuevo modelo de usuario personalizado va a heredar de AbstractBaseUser

{% highlight python %}
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    ciudad = models.CharField(max_length=100, blank=True)
    instagram_nombre = models.CharField(max_length=30, blank=True)

{% endhighlight %}

En tu archivo `settings.py` necesitarás hacer esto:

{% highlight python %}
AUTH_USER_MODEL = 'core.User'
{% endhighlight %}

**Advertencia**: existe una forma segura de referirse a este modelo cuando más tarde creas claves foráneas. Refierete a el modelo usando `settings.AUTH_USER_MODEL` en lugar de directamente a tu modelo de usuario personalizado.

{% highlight python %}
from django.db import models
from django.conf import settings

class BlogPost(models.Model):
    titulo = models.CharField(max_length=100)
    autor = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
{% endhighlight %}

### 4. Creación de un modelo de usuario personalizado extendiendo AbstractBaseUser

Lo más probable es que no necesites esta opción. Es similar a # 3 en que cambia significativamente tu esquema de base de datos (¡así que ten cuidado!) Pero el nuevo modelo hereda de AbstractBaseUser. La gente hace esto cuando su proyecto tiene requisitos específicos relacionados con el proceso de autenticación. Debido a que te permite cambiar la forma en que funciona el proceso de autenticación y puede agregar información adicional relacionada con el usuario, es técnicamente la opción más poderosa, pero es bastante fácil hacer algo que realmente no quieres, por lo que debes tener cuidado .

¡Eso es todo! 👋

[django-docs]: https://docs.djangoproject.com/en/1.10/ref/signals/#post-save
