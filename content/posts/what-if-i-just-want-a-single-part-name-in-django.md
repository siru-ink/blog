+++
title = "What if I just want a single-part name in Django?"
date = "2026-01-27"
+++
Recently, I have been working on a new project to rewrite my homeserver in
[Python3](https://www.python.org) using the
[Django web framework](https://www.djangoproject.com). Hopefully, this will
incentivize me to add more modules onto it, replacing various other app
functionality, than having the server have been written using Rust's
[Rocket web framework](https://rocket.rs) has. And considering this is for my
private family--of currently two--I think the performance will still remain
manageable. Lastly before getting started, I will try to write up a guide on
getting the general framework of a Django project set up soon[^1], so I will not
go into all the details about the default project layouts in this post.

## Does everyone have to have a first and last name?

Django by default expects a user to have a first and last name. In almost all of
my online activities I just go by the moniker siru though, so do I have to make
up a fictionally family name for my digital persona now simply to fit into
Django's world view and be able to use it (without issue) in my homeserver
setup?[^2] Personally, I do not like that idea, so this post is about modifying
the default user model, so that one can still take advantage of all the built-in
authentication/authorization functionallity while modifying the users to only
have a single name field.

## Creating a Model to Suit Your Needs

In order to create a modified vesion of the user model, we need a place to put
this new model. To do this, let us create a new "app" that can hold any models,
views, and admin interface manipulations we want to create. We can do this using
the `startapp` command from the `manage.py` utility. Using `uv` to handle our
virtual environment, we can accomplish this with:

```bash
uv run manage.py startapp accounts
```

or feel free to change the name of the new "app" to anything else, but keep in
mind to make appropriate modifications to the code changes presented in the rest
of this post.

This should give the following directory and file structure:

```
<project-root-dir>/
├── accounts/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── <root-config-dir>/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── ...
```
*generated using https://tree.nathanfriend.com*

This allows us to add a new model for our modified user in the
`accounts/models.py` file. We can add our own new model the same way we would
add any other model by creating a class with instance attributes as:

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

# Create your models here.
class User(AbstractUser):
    first_name = None
    last_name = None
    name = models.CharField(max_length=255, blank=True)
    REQUIRED_FIELDS = []
```

Using the `AbstractUser` class as the superclass for our new model allows us to
reuse existing functionality created for Django's built-in auth users, but it
comes with the downside that we need to drop unwanted baggage. The lines for the
`first_name` and `last_name` are simply removing attributes that already exist
in the superclass, that we do not want to use in our user. Instead, we create a
new unified instance attribute called `name` as a unified for the two. *Note:
While I am only covering how to remove the double name requirement in this post,
you can use the same system to add other things to the user as desired, e.g. a
telephone number field.* A reference for the different database field types that
Django can handle can be found
[here](https://docs.djangoproject.com/en/6.0/ref/models/fields/#field-types).
Lastly, the `REQUIRED_FIELDS` list defines which attributes have to exist for
each user, **in excess** of the username and password. As can be seen in the
implementation for `AbstractUser`, normally, this would be an email address, but
setting this to be an empty list here removes said restriction. I would
encourage you to look at the implementation for `AbstractUser`--after all why
use a JIT programming language if you never peak behind the scenes--for me this
is located at
`<project-root-dir>/.venv/lib/python3.14/site-packages/django/contrib/auth/models.py`,
but it might be a different path for you. Most editors have a *Look up
definition* option of some sort though.

## But how do we register our model to override the built-in model?

The simple answer is that we can simple add a constant variable to our
`<root-config-dir>/settings.py` file, to override the basic auth model as:

```python
AUTH_USER_MODEL = "accounts.User"
```

But before this will run without errors, we need to add some more extra
configuration: First, we need to add our new app to the installed apps for
Django to be aware of it's existence. Do this by pushing it into the stack of
installed apps in the `<root-config-dir>/settings.py` file as:

```python
# ...

INSTALLED_APPS = [
    "accounts.apps.AccountsConfig",
    "django.contrib.auth",
    ...
]

# ...
```

## But wait, this can't be accessed at all!

While those two above steps are *technically* enough to get the new user model
to be reflected in the backend database, we are presumably trying to write a web
application--not an overly complicated database engine--so we need to make our
new user model available in the admin user interfaces.

### Adding my Users to the Admin Interface

All of the admin interface magick is handled in the `<app-root-dir>/admin.py`
files, so for our case this will be the `accounts/admin.py` file. By default,
this file looks like this:

```python
from django.contrib import admin

# Register your models here.
```

Normally, we could simply add a model by adding a line of
`admin.site.register(User)`, but since we are modifying a pre-existing model, we
will need to give Django a bit more information to get things to function
frictionlessly. Namely, we need to override the built in `UserAdmin` display
class. We can do this by subclassing the above mentioned again as:

```python
from .models import User

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    fieldsets = (
        (None, {"fields": ("username", "password")}),
        ("Personal info", {"fields": ("name", "email")}),
        ("Permissions",
            {
                "fields": (
                    "is_active",
                    "is_staff",
                    "is_superuser",
                    "groups",
                    "user_permissions",
                )
            },
        ),
        ("Important dates", {"fields": ("last_login", "date_joined")}),
    )
    add_fieldsets = (
        (
            None,
            {
                "classes": ("wide",),
                "fields": ("username", "password1", "password2", "name", "email"),
            },
        ),
    )
    list_display = ("username", "email", "name", "is_staff")
    search_fields = ("username", "name", "email")
    ordering = ("username",)
```

That is quite the block of code, so let me brake it up into smaller chunks for
easier digestability:

1. We add the decorator `@admin.register(User)` this is simply a shorthand to
   avoid having to make a call to `admin.site.register(User, CustomUserAdmin)`
   after our class definition.
2. `class CustomUserAdmin(UserAdmin)` subclasses the built-in display class for
   the user model in the admin interface. This way we can make some
   modifications before it is actually rendered in the admin interface.
3. All the stuff inside the class. As you will have noticed, this is a purely
   descriptive class. It contains no methods, just definitions for variables.
   All of the heavy lifting of methods is (luckly) already being provided by the
   superclass instead of by us.
    * `fieldsets`: these are the section titles, and database items that are
      shown when we click to modify an existing user in the admin interface. We
      have to redefine it to use the simple `name` instead of the `first_name` &
      `last_name` combination that it would have expected by default.
    * `add_fieldsets`: these are the database items that are requested when
      creating a new user through the admin interface. We can hardly ask the
      administrator to still provide first and last names during setup and then
      just throw them away afterwards.
    * `list_display`: the columns that are shown in this listing of all users in
      the admin interface. We can't have our template try to render columns for
      fields that do not even exist in the database.
    * `search_fields`: the database columns that are matched against the search
      query when using the admin interface's search function.
    * `odering`: this really odd to be called `sort_order` or at least something
      with sort. This defines the column priority used when sorting the entries
      in the admin interface. Surprisingly to me this is not just sorted
      according to the database integer key, so it can be easily modified if you
      prefer to list people by their email address for example.

Now that all this has been set, we can easily add/edit/delete users from our
database using the admin interface.

### What About Non-Admin Users?

For the people who cannot use the Django provided admin interface, we can make
either--same as normal--create our own customs views, forms, templates, etc. to
create the login/logout/register workflows. Alternatively, Django provides
default implementations for all of these via the `django.contrib.auth.urls`
module. So a simple approach would be to simple reference those urls in our
`accounts/urls.py` file to automatically make them available on the server as:

```python
from django.urls import include,path,...

urlpatterns = [
    path('accounts/', include('django.contrib.auth.urls')),
    ...
]
```

Lastly, in the `<project-root-dir>` we will have to create a directory structure
`templates/registration` as:

```
<project-root-dir>/
├── templates/
│   └── registration/
│       ├── login.html
│       ├── logout.html
│       └── registration/password_change_form.html
└── ...
```
*generated using https://tree.nathanfriend.com*

These html template files can then be populated with some basic code to render a
Django form object, which is by default passed under the `form` name as:

```html
<!doctype html>
<html>
    <head>
        <title>Login</title>
    </head>
    <body>
        <h2>Login</h2>
        <form method="post">
            {% raw %}{% csrf_token %}
            {{ form.as_div }}{% endraw %}
            <input type="submit" value="Submit">
        </form>
    </body>
</html>
```
*A sample `login.html` file*

Note: Even though the form has multiple subfields, simple referencing the
`{% raw %}{{ form }}{% endraw %}` will render all of them automatically. The `as_div` puts each input
& label pair into its own `div` container.

> [!Tip]
> If you made it all the way to here, please drop me an email at
> "mail [at] siru [dot] ink". I would be interested to know whether any real
> human beings ever read this, or whether the trafic that is recorded for this
> site is just GoogleBot, and OpenAI, Antrhopic, etc. crawlers.

[^1]: I would consider myself notoriousely bad at starting a bunch of setup
      guides, and then never polishing them up enough to want to publish them.
      (My drafts folder on this blog is way too full.) Hopefully having
      published this out in the public will actually incentivize me to finish
      this setup guide for once.

[^2]: Just so nobody calls me out later for hating on the Django web project, I
      feel the need to add this footnote. By no means, do I disagree with the
      Django developer team's decision to make a last+first name combination the
      default setup. I think for most usecases this will make the most sense and
      is a sensible default. Furthermore, they do themselves provide all the
      necessary documentation to figure out how to modify these built-in models,
      so they are doing a great job. This post is simply to be a more concise
      version of solving this exact issue without having to peace together all
      the various pieces of the documentation needed to make this work.
