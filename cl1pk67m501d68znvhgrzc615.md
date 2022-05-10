## How to prevent “RecursionError: maximum recursion depth exceeded” when using Django Signals.

As we grow in web application development, it comes to a point when we want to be able to carry out some tasks just before saving an object to the database or right after saving an object to the database.

In Django, this can be done using a feature called Signals. According to the [documentation](https://docs.djangoproject.com/en/2.2/topics/signals/), Signals allow certain _senders_ to notify a set of _receivers_ that some action has taken place and that they’re especially useful when many pieces of code may be interested in the same events.

An example of an instance when using signals can be helpful is when you are trying to create an object after saving another one. Like when you try to create a Profile object immediately after creating a User object. Following the steps in the documentation would work smoothly.

However, in a case whereby you need to make changes to an object/objects of the same model, then you could have a challenge. In our example, we have a Favorites model and a Category model. A Favorite object is an item that has a name, category and rank in numbers. A Category object is just a category a Favorite object can belong to.

```
class Category(models.Model):  
name = models.CharField('Name', max\_length=30)  
description = models.TextField('Description', blank=True)
```
  
 

```
class Favourite(models.Model):  
title = models.CharField('Title', max\_length=20)  
description = models.TextField('Description', blank=True)  
rank = models.SmallIntegerField('Ranking',)  
category = models.ForeignKey(Category, on\_delete=models.CASCADE, related\_name='categories')
```

Above, we have the 2 models, Category and Favorites. In this little project, we want to adjust the ranks of our Favorites if there is a new Favorite with the same category and rank that already exists. For example, if we have a Favorite with name ‘Jordans’ in the ‘Shoes’ category with a rank of 1 and we want to add another Favorite with the name ‘Air Force One’ in the ‘Shoes’ category and rank of 1. We would want to update the rank of the ‘Jordans’ to 2 so that we can have the ‘Air Force One’ in rank 1. This way we would always have a unique Favorite every time.

One way to do this would be to add ‘unique-together’ as a meta property in the Favorite model and set it to rank and category. However, what this would do is to raise an error when we have a new Favorite with the same category and rank but would not make any changes. To resolve this, we can use a pre-save signal that would check the Favorite, before its saved if there already exists a Favorite with the same category and rank. If this is true, we then update the all existing Favorites with same category and rank that is equal or greater than that of the incoming Favorite to have 1 added to their rank.  

```
def save\_favorite(sender, instance, \*\*kwargs):
   fav = instance
   try:
       favs = Favourite.objects.filter(rank\_\_lte=fav.rank, category=fav.category)
       for f in favs:
           f.rank += 1
           f.save()
       print('ending')
   except Favourite.DoesNotExist:
       passpre\_save.connect(save\_favorite, sender=Favourite)
```

  
But then, when you do this, you get this error:  
RecursionError: maximum recursion depth exceeded while calling a Python object  
  
This error occurs because we have modified the process of saving a Favorite object — the `f.save()` method will always call the `save_favorite()` and will result in an unending recursion which raises the above error.

To resolve this error, we will follow a different approach to saving our Favorite objects.

```
def save\_favorite(sender, instance, \*\*kwargs):
   fav = instance
   Favourite.objects.filter(rank\_\_lte=fav.rank, category=fav.category).update(rank=F('rank') + 1)pre\_save.connect(save\_favorite, sender=Favourite)
```

Now, we see that we are no longer using the `save()` method which will result in the recursion. The `update()` works very well when you want to make updates to an object or objects of a model as we have done for the Favorite model.

I hope this article was helpful. Please comment below if you have any questions and I will reach out as soon as I can.

**This post was written by [Ahmad Bilesanmi](https://www.linkedin.com/in/bilesanmi-olorunfemi/)**