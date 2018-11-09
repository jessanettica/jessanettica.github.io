---
layout: post
title:  "Cómo utilizar una API pública"
date:   2017-05-19 9:02:22 -0800
tags: code
image_sliders:
  - slider8
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
photos: ["/img/lipstick_above.jpg", "/img/makeup_brush.jpg"]
---

{% include slider.html selector="slider8" %}

API significa Application Program Interface. La definición "oficial" de wikipedia es "un conjunto de definiciones, protocolos y herramientas para crear software para aplicaciones". Mi definición es una interfaz web que toma solicitudes y devuelve los datos (por ejemplo, json, xml, etc) que pido. Hay muchos tipos de APIs, pero este post se trata del uso de APIs públicas. El propósito de las API públicas es permitir que aplicaciones y proyectos externos consulten y obtengan datos.

Hay un montón de APIs públicas realmente geniales con una hermosa documentación fácil de usar que nos permiten crear aplicaciones usando datos del mundo real. Por ejemplo, se puede utilizar la API de Eventbrite para obtener eventos que ocurren en diferentes ciudades, la API de Instagram para obtener posts etiquetados #findesemana, la API de Google Maps para mostrar tus rutas de bici favoritas, ... bla bla bla. Ya tienes la idea. Tu puedes crear cosas impresionantes! Sí, también puedes crear cosas impresionantes sin usar una API pública, solo depende de lo que sea tu proyecto. Te sorprendería cuántas cosas usan APIs públicas. ¿Has visto el botón "Iniciar sesión con Facebook" como opción al tratar de registrarte o iniciar una sesión en un sitio web? Eso está utilizando la API de Facebook.

Esta es una lista super simplificada de pasos para usar una API pública para alguno de tus proyectos:

1. Obtén un token de acceso de la API pública que estás utilizando
2. Envía una solicitud a uno de los puntos finales de la API con tu token de acceso
3. Obtén los recursos que solicitaste en la respuesta de la API
4. Analiza la respuesta y utiliza los datos como desees en tu aplicación

Nota: Voy a utilizar Python y la biblioteca “requests” en mi ejemplo, pero puedes usar cualquier lenguaje y cualquier biblioteca http que quieras. Los pasos siguen siendo los mismos.

Bueno, entremos en los detalles de cada paso.

1

La mayoría de las API públicas hoy en día tienen una documentación realmente bella que te dirá paso a paso cómo obtener un token de acceso. Puedes pensar en un token de acceso como una contraseña secreta que se le da a la API para que sepa que está totalmente bien darte los datos que pides. Normalmente hay dos tipos de fichas de acceso que puedes obtener.

Un token personal es para cuando estás construyendo algo solo para ti o para una sola organización. Puedes usar el token personal para representarte a tí mismo. Con este token normalmente sólo puedes obtener tus propios datos. Por ejemplo, puedes pedir tus propias fotos de Instagram pero no puedes conseguir una foto de las vacaciones del 2014 de ora persona.

La otra opción es el flujo de token OAuth. Esto se usa para acceder a la API en nombre de otros usuarios que no sean tu. ¿Te acuerdas de la opción para iniciar una sesión o registrarte en un sitio de web a través de la opción de Facebook que mencione antes? Si alguna vez lo has utilizado, sabrás que cuando intentas iniciar la sesión, aparece una ventana pidiéndote que autorices el inicio de sesión. Básicamente, te está preguntando si esta bien que inicie una sesión en Facebook de tu parte. Cuando la autorizas, estás diciendo que sí, puedes acceder a la API de Facebook en mi nombre para iniciar una sesión en tu sitio usando mis credenciales de Facebook. Esto no es necesariamente más difícil que el token personal, sólo requiere más pasos de implementación. Para simplificar mi ejemplo, voy a usar un token personal.

Ok, si vas a la página de la [API][api-page], baja a la parte de la pagina donde dice "Getting a Token". Si haces clic en Token personal, te llevará a la página de aplicaciones.

Haz clic en "Create A New App" (Crear una nueva Aplicación), luego llena la forma que aparece y BAM! Te da una ficha (una serie de letras y números). El formulario solicita el nombre de tu aplicación. No te preocupes y pon lo que sea. Puedes regresar después y obtener otra token si se te ocurre un nombre perfecto para tu applicacion/proyecto.

2

https://www.eventbriteapi.com/v3/users/me/?token=MYTOKEN

3

{% highlight javascript %}
{
    "emails": [
        {
            "email": "jessica-email-hello@gmail.com",
            "verified": false,
            "primary": true
        }
    ],
    "id": 1234567,
    "name": "Jessica Anette",
    "first_name": "Jessica",
    "last_name": "Lopez"
}
{% endhighlight %}

4

La respuesta de la API Eventbrite es JSON. Puedes usar la respuesta como quieras. Por ejemplo, si tienes una pequeña sección para el perfil de usuario en tu aplicación, puedes usar los datos de arriba para mostrar el nombre y correo electrónico.

Echemos un vistazo a un ejemplo más realista de lo que podrías recibir de la API cuando creas una aplicación. El código siguiente combina los pasos 2, 3 y 4. Lo tomé de una función en un proyecto que hice hace un tiempo (si te interesa puedes ver todo el programa [aquí][eventbrite-ex]).

{% highlight python %}
def sf_experience(category):
    token = os.environ.get('EVENTBRITE_TOKEN')
    """Show Eventbrite experiences available in SF"""
    params = {'token': token, 'venue.city': 'San Francisco', 'categories': category}
    r = requests.get('https://www.eventbriteapi.com/v3/events/search/', params=params)
    events = r.json()['events']

    for event in events:

        print "Currently working on event:", event

        event_name = event.get('name').get('text')
        event_description = event.get('description').get('text')
        event_start_datetime = datetime.datetime.strptime(event.get('start').get('local'), "%Y-%m-%dT%H:%M:%S")
        event_end_datetime = datetime.datetime.strptime(event.get('end').get('local'), "%Y-%m-%dT%H:%M:%S")
        event_category = category
        event_city = "San Francisco"
        event_id_sf = event.get('id')
        venue_id_sf = event.get('venue_id')

{% endhighlight %}

Dicen que nunca deberías exponer tu token de acceso (secreto!), Por lo que lo configuré como una variable externa y obtuve el token con `os.environ.get`, pero cuando estás jugando en tu terminal, está bien. Como puedes ver, obtengo mi token y luego la pongo en los parámetros que envío con la solicitud a la API de Eventbrite para eventos en San Francisco. Una vez que tengo la respuesta, analizo el JSON para obtener información más específica sobre el evento.

Jessica 👋

[api-page]: https://www.eventbrite.com/developer/v3/api_overview/authentication/
[eventbrite-ex]: https://github.com/jessanettica/Andarography
