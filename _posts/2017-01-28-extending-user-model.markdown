---
layout: post
title:  "Extending Django's User Model"
date:   2017-01-28 09:55:21 -0800
categories: python
image_sliders:
  - slider1
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
photos: ["img/beach.png"]
---

{% include slider.html selector="slider1" %}

Django’s built-in user authentication system is awesome. It handles both authentication and authorization, but for many of your projects you’ll want to extend and customize it. One of the main reasons you will want to customize the built-in system is to store more data related to the User. For example, you might want to store the user’s city, or her instagram handle. There are different ways to extend the User Model, and there is no general “best” or “worst” option. The best option will depend on your project and how far along in building it you are. Ok, so let's get to it.

If you are mid-way through your project options #1 or #2 are the better options. They both extend the default User model without substituting it with your own model.

### 1. A One-to-One Link with a User Model(Profile)

Best option if you are not at the very beginning of starting your project and you need to store extra information about the existing User Model that’s not related to the authentication process. Usually people that do this call the table UserProfile, but you can do you and call it whatever you want!

{% highlight python %}
from django.contrib.auth.models import User

class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    city = models.CharField(max_length=100, blank=True)
    insta_handle = models.CharField(max_length=30, blank=True)
{% endhighlight %}


Now in the shell you can do:

{% highlight shell %}
>>> u = User.objects.get(username='jessica')
>>> jessica_instagram_handle = u.userprofile.insta_handle
{% endhighlight %}

So as you can see, it is not a special model in any way. It's just a normal Django model that happens to have a one-to-one relationship to the User Model through a OnetoOneField and it will have it’s own database table. Because UserProfile is just another model, it will not automatically be updated when a new user is created. To do this automatically you will need to use Django signals to listen for the User’s post_save event and update the UserProfile instance when it happens (the Django docs are [magical and beautiful][django-docs])

As a side note, the code above is for Dango 1.10. If you are like me and have been using Django 1.8 or before, `on_delete` can now be used as the second positional argument (previously it was typically only passed as a keyword argument). It’s not required, but it will be in Django 2.0 so mind as well start using it and checkout what the options for it are. I used CASCADE here which tells Django: “Hey, when the referenced object is deleted, also delete the objects that have references to it, ok? Thanks.”

An important thing to think about is that using a related model results in additional queries or joins to retrieve the data. You can optimize your queries, but most times you try to access related data, Django will make an additional query. This is why if you are at the very beginning of a project a custom User model might be a better option (see option  #3).

Don’t forget that if you want to add your new UserProfile model’s fields to the user page in the admin you need to register it! In your app's `admin.py` add it to to UserAdmin class which is registered with the User class.


### 2. Using a Proxy Model

This should be your game plan when you don’t need to store extra info in the database but want to add extra methods. This is model inheritance but it doesn’t create a new table in the database; the existing database schema is not affected.

{% highlight python %}
from django.contrib.auth.models import User
from .managers import BloggerManager

class Subscriber(User):
    objects = SubscriberManager()

    class Meta:
        proxy = True
        ordering = ('first_name', )

    def get_subscription_plan(self):
        ...
{% endhighlight %}

This changes the default ordering to be by first_name and assigns a custom Manager to Subscriber. `get_subscription_plan` is just an an example method. You can make whatever method you need for your project.

After this `User.objects.all()` and `Subscriber.objects.all()` will query the same database table, which is kind of cool. The only difference is in the behavior defined for the Proxy Model.

If you are at the beginning of your project, the next two options are the better ones. They are not as common because they significantly change your database schema and also because people usually don’t know how they’ll want to customize the built-in system at the very beginning. So if you have an epiphany about changing user authentication in the middle of your project, it's cool but go back and look at #1 or #2.


### 3. Create a Custom User Model Extending AbstractUser

This is the best option if you are down with how Django handles auth but you want to add some extra info directly in the User model, without having to create an extra class (like in #2). In an ideal world where you know what extra info your User model will need at the beginning of a project, this is the best option. It is simple and avoids the complexity of setting up #4, and it prevents the mental acrobatics you will have to do to remember how your app’s system works and why later on if you do #1 or #2.

Your new custom User model is going to inherit from AbstractBaseUser

{% highlight python %}
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    city = models.CharField(max_length=100, blank=True)
    insta_handle = models.CharField(max_length=30, blank=True)

{% endhighlight %}

In your `settings.py` file you will need to do this:

{% highlight python %}
AUTH_USER_MODEL = 'core.User'
{% endhighlight %}

**Warning**: there is a safe way to reference this model when you are later creating foreign keys by referring to `settings.AUTH_USER_MODEL` instead of directly to your custom User model.

{% highlight python %}
from django.db import models
from django.conf import settings

class BlogPost(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
{% endhighlight %}

### 4. Create Custom User Model Extending AbstractBaseUser

You most likely don’t need this option. It is similar to #3 in that it significantly changes your database schema (so be careful!) but the new model inherits from AbstractBaseUser. People do this when their project has specific requirements relating to the authentication process. Because it allows you to change how the authentication process works and you can add extra info related to the User, it is technically the more powerful option, but it’s pretty easy to do something you actually don't want, so you have to be careful.

That’s it! 👋

[django-docs]: https://docs.djangoproject.com/en/1.10/ref/signals/#post-save
