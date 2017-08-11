---
layout: post
title:  "Using a Public API"
date:   2017-05-18 8:25:10 -0800
categories: python
image_sliders:
  - slider7
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
---

{% include slider.html selector="slider7" %}

API stands for Application Program Interface. The “official” definition from wikipedia is “a set of subroutine definitions, protocols, and tools for building application software.” My definition is a web interface that takes requests and gives back the data (ex. json, xml, etc) I ask for. There are many kinds of API’s, but this post is about using public APIs. The purpose of public APIs is to let external apps and projects query and get data.

There are tons of really cool public API's with beautiful user-friendly documentation that enable you to build applications on top of real-world data. For example, you can use the Eventbrite API to get events happening in different cities, the Instagram API to get posts tagged #weekendvibes, the Google Maps API to show your favorite hiking trails, ...blah blah blah. You get the idea. You can build awesome things. Yes, you can also build awesome things without ever touching a public API, it just depends on what your project is. You’d be surprised how many things use public APIs. Have you seen that “Sign In with Facebook” button as an option when signing in to a website? Yup, that’s using the Facebook API.

Here’s a super simplified list of steps to use a public API from your app or pet project:

1. Get an Access Token from the public API you are using
2. Send a Request to one of the API's endpoints with your access token
3. Get the resources you requested in the API's response
4. Parse the response and use the data however you want in your app

Disclaimer: I’m going to use Python and the requests library in my example, but you can use whatever language and http library floats your boat. The steps are still the same.

Okay, let’s break down those steps.

1

Most public API’s nowadays have really nice documentation that will tell you step by step how to get an access token. You can think of an access token as a secret password you give the API so that it knows it’s totally fine to give you the data you ask for. There are usually two kinds of access tokens you can get.

A personal token is for when you are building something only for you or a single organization. You can use the personal token to represent yourself. With this token you can usually only get your own data. For example, you can ask for your own Insta pics but you can’t get someone else’s photo from their vacation last week.

The other option is the OAuth Token flow. This is to access the API on behalf of users other than yourself. You know that log in through facebook option I mentioned earlier? If you’ve ever used it you know that when you try to log in you get a little modal asking you to authorize logging in. Basically it’s asking if it’s cool to Log In to Facebook for you. When you authorize, you are saying yes, you can hit the Facebook API on my behalf to log in to your site using my facebook credentials. This isn't necessarily harder than the personal token, just requires a couple more implementation steps. For the sake of simplicity in this example, I'm just going to use a personal token.

Ok so, if you go to the [API page][api-page] then scroll to the “Getting a Token” section. If you click on Personal Token it will take you to the apps page.

You click on “Create A New App”, then you fill out the form that pops up and BOOM! It gives you a token (a series of letters and numbers). The form asks for an App name. Don’t stress and just put down whatever. You can always get another token if you decide you have a brilliant idea for your startup and have THE perfect name.

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

The Eventbrite API’s response is JSON. You can use the response however you want. For example, if you had a little user profile section in your app you can use the above data to display your name and email.

Let’s look at a more realistic example of what you might have when you’re building out an app. The following code combines steps 2, 3, and 4. I took it from a view function in a side project I made a while back (you can look at the whole thing [here][eventbrite-ex]).

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

They say that you should never hard code your (secret!) access token, which is why I set it as an environment variable and got it with `os.environ.get` but when you’re just playing around in your terminal it’s fine. So here you can see, I get my token and then put it into the parameters I send with the request to the Eventbrite API for events in San Francisco. Once I have the response, I parse the JSON to get more specific info about the event.

Jessica 👋

[api-page]: https://www.eventbrite.com/developer/v3/api_overview/authentication/
[eventbrite-ex]: https://github.com/jessanettica/Andarography
