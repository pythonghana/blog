---
title: "Programming the web with Python and Django"
datePublished: Mon Mar 10 2025 23:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cmbi7rh1u000f09l1bx93fw8k
slug: programming-the-web-with-python-and-django
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1741650902944/95981b19-f59a-45b0-96bb-face807c9484.jpeg
tags: python, django, web-development

---

Python is versatile, cutting its way into countless applications and uses and crafting dynamic web experiences is where it truly shines. But on the real, programming the web with Python can feel like navigating a jungle for beginners (and even some seasoned developers!).

Staring at the terminal in front of you and wondering what next - turning that app idea into reality? We've all been there so has many Python learners stumbled when it came to structuring a Django project from scratch.

Going back to the basics, we will be building a real-world application: Community Hub – a forum-like platform where members can post blog articles, learn the process of building Django apps, and grow. Think user sign-in/-up, engaging blog posts, and personalized profiles!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741687291030/b5b92525-b67f-4169-b872-b9dc17ce2090.png align="center")

**Project Setup, Step-by-Step**

First things first: the blueprint. Setting up a boilerplate ensures a solid foundation for our Community Hub project.

```bash
# Create the project
django-admin startproject communityhub

# Navigate into the project directory
cd communityhub

# Create the app
python manage.py startapp app
```

**Building our data Models**

Next, we'll dive into defining the structure of our data with Django's powerful models.

```python
# app/models.py
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(blank=True)

    def __str__(self):
        return self.user.username

class BlogPost(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

Let’s add `'app'` to `INSTALLED_APPS` in `communityhub/`[`settings.py`](http://settings.py):

```python
# communityhub/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app', # Add your app here
]
```

And then we run our migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

**Writing Our Views**

Writing our view logics that handles user requests, process data and returns a response.

```python
# app/views.py
from django.shortcuts import render
from .models import BlogPost

def blog_list(request):
    posts = BlogPost.objects.all()
    return render(request, 'app/blog_list.html', {'posts': posts})
```

**Matching our URLs to Views**

We will have to map our URLs to specific views, to help access and navigate our application.

```python
# communityhub/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app.urls')), # Include app's urls
]
```

And now, we create `app/`[`urls.py`](http://urls.py):

```python
# app/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.blog_list, name='blog_list'),
]
```

**Templates to render our Views**

Finally, we create a text-document know as templates to display our data and create a user-friendly interface. It *<mark>“can generate any text-based format (HTML, XML, CSV, etc.).”</mark>*

Create a `templates/app` directory and a `blog_list.html` file inside it:

```xml
<!DOCTYPE html>
<html>
<head>
    <title>Blog Posts</title>
</head>
<body>
    <h1>Blog Posts</h1>
    <ul>
        {% for post in posts %}
        <li>
            <h2>{{ post.title }}</h2>
            <p>By: {{ post.author.username }}</p>
            <p>{{ post.content }}</p>
            <p>Created: {{ post.created_at }}</p>
        </li>
        {% endfor %}
    </ul>
</body>
</html>
```

Let’s not forget to configure the `TEMPLATES` setting in `communityhub/`[`settings.py`](http://settings.py):

```python
# communityhub/settings.py
import os

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')], # Add this line
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

Now, we run the development server:  

```bash
python manage.py runserver
```

Our application can be accessed via [`http://127.0.0.1:8000/`](http://127.0.0.1:8000/) in your browser to see the blog list but we will have to create some blog posts in the admin panel first.

In our next session, **"**[**Django Deep Dive**](https://www.meetup.com/python-ghana/events/305639121/?utm_medium=referral&utm_campaign=share-btn_savedevents_share_modal&utm_source=link)**,"** we will explore advanced ORM techniques, delve into authentication best practices, and master admin customization.