---
layout: post
title:  "Django QuerySets 101"
date:   2017-02-28 09:55:21 -0800
categories: python
image_sliders:
  - slider3
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
photos: ["img/berries.jpg", "img/candy.jpg", "img/citrus.jpg"]
---

{% include slider.html selector="slider3" %}

*What is a Queryset? Is it a list?*

Umm... tempting, but don’t think of it as a list.

{% highlight shell %}
>>> p = BlogPost.objects.all()
>>> type(p)
>>> <class 'django.db.models.query.QuerySet'>
{% endhighlight %}

See? Not a list. Think of a QuerySet as a collection of objects of a given Model. It’s basically a representation of a number of rows in the database that can be filtered by a query.

&nbsp;

*How can I play with Django queries for my project?*

In your project folder type:

{% highlight shell %}
>>>python manage.py shell
{% endhighlight %}

Depends on the models you have, but for example if you have  BlogPost model you can do:

{% highlight shell %}
>>>BlogPost.objects.all()

Traceback (most recent call last):
      File "<console>", line 1, in <module>
NameError: name 'BlogPost' is not defined
{% endhighlight %}

Oops we forgot to import the model.

{% highlight shell %}
>>>from blog.models import  BlogPost
{% endhighlight %}

Now try that again and it will work :)

&nbsp;

*How do I make an object?*

{% highlight python %}
# this is getting a user we are going to save as the author of the BlogPost object we are going to create
me = User.objects.get(username='jessanettica')
# now we make the BlogPost object
BlogPost.objects.create(author=me, title="Sample title", text="Testing. Testing.Hello you there?")
Post.objects.all()
{% endhighlight %}

&nbsp;

*What if I only want to see Blog posts by a specific person?*

Get the user you want (in this case I’m just going to use the one we already have) and do something like this:
{% highlight shell %}
>>> Post.objects.filter(author=me)
[<Post: Sample title>, <Post: Post number 2>, <Post: My 3rd post!>, <Post: 4th title of post>]
{% endhighlight %}
Nice.

&nbsp;

*How can you chain multiple filters?*

{% highlight python %}
# Return all blog posts posted after 2014, except for
# ones written by Jessica Lopez. Order them by
# author name, then chronologically, with the most recent
# ones first.
BlogPost.objects.filter(year_posted__gt=2014) \
            .exclude(author='Jessica Lopez') \
            .order_by('author', '-year_published')
{% endhighlight %}

&nbsp;

*My friend said Querysets are awesome because they are lazy. What does she mean?*

When you make a Queryset, you are not necessarily hitting the database. You can chain multiple filters and Django won't actually hit the database until the Queryset is evaluated. More on what evaluated actually means later. For now check this out:

{% highlight shell %}
>>> p = BlogPost.objects.filter(headline__startswith="How to")
>>> p = p.filter(pub_date__lte=datetime.date.today())
>>> p = p.exclude(body_text__icontains="food")
>>> print(p)
{% endhighlight %}

How many times do you think Django hit the database above? It looks like maybe three times, right? Django actually hit the database only once, at the last line when the queryset is printed. Cool because it’ll make things faster without hitting the database a bunch of unnecessary times.

&nbsp;

*Soooo when are Querysets evaluated if not when they are filtered?*

A QuerySet can be created, filtered, sliced, and passed around without actually hitting the database, but to use this to your advantage you need to know when they are actually evaluated.

#### Iteration

A QuerySet is iterable. It executes its database query the first time you iterate over it.

{% highlight shell %}
>>>for post in BlogPost.objects.all():
...    print(post.title)
{% endhighlight %}

#### Slicing (kind of)

A QuerySet can be sliced like a list, using Python’s list-slicing syntax (but remember, a QuerySet is NOT a list). Slicing an unevaluated QuerySet usually returns another unevaluated QuerySet, but Django will hit the database if you use the “step” parameter of slice syntax, and will return a list. Slicing a QuerySet that has been evaluated also returns a list.
Kind of a bummer, but when you slice an unevaluated QuerySet and return  another unevaluated QuerySet, modifying it more ( adding more filters, or changing the ordering, etc.) is not possible.

#### repr()

A QuerySet is evaluated when you call `repr()` on it. This is for your sake so that in the Python interactive interpreter (the shell), so you can immediately see your results when using the API interactively. `repr()` is what is called to get the “official” string representation of an object.

Random side note: I actually ran into this at work the other day. I was checking a query in the shell and it wasn’t working because the name of a field had changed and I was trying to get a queryset of objects I needed to change the field for...migrations were involved...anyway. It didn’t work in the shell because it was calling `repr()` to show them to me, but I wrote the same thing in a script, ran the script, then checked the data and it worked perfectly. Not super important, just be awake of it as a thing that can happen.

#### len()

A QuerySet is evaluated when you call `len()` on it. You probs guessed it, but it  returns the length of the result list. If you are doing this because you want to know the number of records in the set, ditch `len()` and use `count()`.
For example:

{% highlight shell %}
>>>BlogPost.objects.filter(headline__startswith="How to").count()
{% endhighlight %}
That will give you the number of Blog Posts whose title starts with “How to.”

#### list()

You can force evaluation of a QuerySet by calling `list()` on it. Most of the time you can avoid this and don’t actually need to do it. Just saying.
{% highlight shell %}
>>>entry_list = list(Entry.objects.all())
{% endhighlight %}

#### If statements or bool()

Using a QuerySet in a boolean context, like using `bool()`, or, and or an if statement, will cause the query to be executed. If there is at least one result, the QuerySet is True, otherwise False.

&nbsp;

*Ok so now I know when they are evaluated but don’t know what being evaluated even means.*

Fair. When a QuerySet is evaluated it means that all the matching rows are fetched from the database are fetched from the database and converted to Django models.

&nbsp;

*Are QuerySets saved?*

Yeah! Well, kinda. They are cached (stored in cache memory) in certain situations.
{% highlight python %}
blog_posts = BlogPost.objects.filter(body_text__icontains="food")  # can you tell I need a snack?
# On the next line the query is executed and cached
for post in blog_posts:
    print(post.title)
# The cache is used next time you iterate and there is no hit to the db.
for post in blog_posts:
    print(post.content)
{% endhighlight %}

Same goes for any time a QuerySet is evaluated (see the handy list above). Say you use an if statement instead of a for loop first.

{% highlight python %}
blog_posts = BlogPost.objects.filter(body_text__icontains="food")
# The `if` statement evaluates the queryset
if blog_posts:
    # The cache is used for subsequent iteration.
    for post in blog_posts:
        print(post..title)
{% endhighlight %}

But what if you don’t want to fetch all the blog posts? You just wanted to check if there was at least one match, right? In that case, using exists() would be better.

{% highlight python %}
blog_posts = BlogPost.objects.filter(body_text__icontains="snacks")
# `exists()` check avoids putting everything in the queryset cache
if blog_posts.exists():
    # No rows were fetched from the database. That means we saved
    # saved bandwidth and memory.
    print("OMG a post on snacks!")
{% endhighlight %}

&nbsp;

Ok, that’s it. I’m off to get a snack, but keep an eye out for a future post on query optimization. 👋
