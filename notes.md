# A file for storing notes for this project.

## Vagrant

### Creating a Vagrantfile

`vagrant init <type-of-os>`

In our case, `<type-of-os>` is `ubuntu/bionic64`. So, we'll run the following command:

`vagrant init ubuntu/bionic64`

This command will create a vanilla ubuntu image based on the standard ubuntu. We'll, of course, need to modify this file according to our needs.

First thing is to set the `config.vm.box_version` to a specific version to shield against any breaking changes introduced in the newer versions. In our case, it is set to:

`config.vm.box_version = "~> 20200304.0.0"`

Next, the `config.vm.network` maps a specific port from our local machine to the machine on our server.
In our case, it is:

`config.vm.network "forwarded_port", guest: 8000, host: 8000`

We'll be running our app on port 8000, and we want to make this port accessible from our host machine.
`host` machine is the one on which we are running the development server, and the `guest` machine is the development server itself. By default, the ports are not accessible on the guest machine, necessitating the need for the above configuration. Now, we can access the ports by going `localhost:8000`, and it will automatically map the connection to our `guest` machine, the development server.

Next, we have a `provision` block:

`config.vm.provision "shell", inline: <<-SHELL`

This allows us to run scripts when we first create our server. Let's look at the commands of this script line by line.

`systemctl disable apt-daily.service`

`systemctl disable apt-daily.timer`

Above two lines disable the auto update, which conflicts with the third line (coming up) of the script.

`sudo apt-get update`

`sudo apt-get install -y python3-venv zip`

Third line will update the repository with all of the available packages, so that in the fourth line, we can install `python3-venv` and `zip`. `python3-venv` is a virtual environment package needed to isolate our repository dependencies. `zip` will allow us to create `.zip` files.

`touch /home/vagrant/.bash_aliases`

Fifth line creates a `.bash_aliases` file, and then using:

```
if ! grep -q PYTHON_ALIAS_ADDED /home/vagrant/.bash_aliases; then
  echo "# PYTHON_ALIAS_ADDED" >> /home/vagrant/.bash_aliases
  echo "alias python='python3'" >> /home/vagrant/.bash_aliases
fi
```

we set `python3` as the default python for the user. Now, we don't need to type `python3` and can simply type `python`.

### Starting our Vagrantbox

Now that we have configured our `Vagrantfile`, we can start a Vagrantbox on our machine.

NOTE: Since the script provided in the `Vagrantfile` contains bash commands, we need the git bash terminal to run that file on Windows.

`vagrant up`

Use the above command to start the Vagrantbox.

After our Vagrantfile has been run and Vagrantbox has been created, we can now use our development server by connecting to it. We need to connected to that Vagrantbox using SSH since it is a completely isolated box from our machine, on a different OS.

`vagrant ssh`

You can know that you're on the Vagrantbox machine with Ubuntu when you're terminal name changes. (For me, it is `vagrant@ubuntu-bionic:~$`).

To disconnect from the machine, simply type

`exit`

And you'll be taken to your machine OS terminal. When you're in the Vagrantbox, any command that you type will be run on the guest OS instead of your local machine.

That's how you connect to the server with Vagrant.

### How Vagrant works

Because the development server is a virtual machine on our computer, by default, the file system is not synchronized. All of the files on our development server are different from the files on our local machine.
Vagrant works by creating a synchronized directory on our vagrant server that updates itself with all of the files in our local project every time we make changes.

Once we're connected to the Vagrant server, type

`cd /vagrant`

This will switch us to the Vagrant directory on our server. Everything in this `vagrant` directory is synchronized with everything in our project folder. If we create any file in this `vagrant` directory, that file will appear in our project folder as well. This synchronization works both ways. Any change in the `vagrant` directory is reflected in our project folder, and any change in the project folder is reflected in the `vagrant` directory.

To run our project directory python code in the Vagrant server, simply `cd` to the vagrant directory, and type

`python file_name.py`

## Python

### Creating a python virtual environment on our Vagrant server

First, connect to the Vagrant server using:

`vagrant ssh`

`cd` into the `vagrant` directory using:

`cd /vagrant`

Now, create a python virtual environment using:

`python -m venv ~/env`

`~/env` is the path where the virtual environment is to be created.

Above command will create a new directory in our Vagrant server's home directory and create our python virtual environment there. The reason this `env` folder has been created in the home directory and not directly in the `/vagrant` folder is because we do not want it to synchronize with our local machine. So, if we ever want to destroy and recreate a vagrant server from scratch, we can do it with a fresh python virtual environment.

### How virtual environments work

Virtual environments need to be activated and deactivated. When it's activated, all of the dependencies required for our python application will be pulled from that virtual environment instead of the globally installed dependencies.

To activate a virtual environment, use:

`source <path-to-the-activate-script-in-our-virtual-environment>`

In our case, it will be:

`source ~/env/bin/activate`

Once our virtual environment has been activated, it will show up in the start of our terminal prompt as

`(name-of-our-virtual-environment-directory) vagrant@ubuntu-bionic:/vagrant$`

Since we named our virtual environment `env`, thus, in our case, it is:

`(env) vagrant@ubuntu-bionic:/vagrant$`

If the virtual environment is active, then to deactivate it, simply type:

`deactivate`

And the terminal prompt will return back to normal.

### Installing the required Python packages in our virtual environment

It is a best practice to list all of the requirements for a particular Python app in a file called `requirements.txt`. The syntax for specifying a particular requirement looks like:

`package-name==version-number`

If the `version-number` is not specified, then the latest version of that package is installed.

After listing the app requirements in `requirements.txt`, go to the `/vagrant` directory in the vagrant server (whilst the virtual environment is activated), and type

`pip install -r requirements.txt`

Above command will install all of the packages (of the version specified or the latest one if no version specified).

### Creating a new Django Project

Ensure you're in the `/vagrant` directory on the Vagrant server and that the virtual environment is active,
then we can create a django project using:

`django-admin.py startproject <project-name> <path-where-the-project-is-to-be-created>`

In our case:

`django-admin.py startproject profiles_project .`

The last argument (path) is optional. If we don't specify one, then `django-admin` will create a subfolder with the name of our project name and add the files there. We do not want that. We want the files in our vagrant root folder, and due to its synchronizing nature with our project folder, also in our root project directory.

The above command will create the following files/directories in our root folder:

```
manage.py
profiles_project/
  |init.py
  |settings.py
  |urls.py
  |wsgi.py
```

We'll take a look at these files/directories later.

### Creating a new Django App

Now that we have created a Django project, we need to create a Django app within that project for our profiles API.

NOTE: A Django project can consist of one or more Django apps within that project that we can use to separate different functionality within our project. In our case, we'll only be creating one app for our profiles API.

To create an app within a Django project, we use:

`python manage.py start_app <app-name>`

In our case, it is:

`python manage.py start_app profiles_api`

`manage.py` is the file that was created when we started our project. It contains different functionality to help us manage our project.

Above command will create the following directory in our root directory:

```
profiles_api/
migrations/
  |__init__.py
__init__.py
admin.py
apps.py
models.py
tests.py
views.py
```

We will take a look at these files later.

### Enabling our Django app

Once our app has been created, we need to enable it. To do this, go to the `profiles_project` directory in our root folder and open the `settings.py` file. This is the configuration file for our django project. Search through this file for a block called `INSTALLED_APPS`. This block contains a list of all the apps that are to be used in our project. We need to add the following entries there:

```
'rest_framework', # to use the functionality provided by the djangorestframework
'rest_framework.authtoken', # allow us to use the authentication token functionality that comes with DRF
'profiles_api', # our app
```

Adding our app in the `INSTALLED_APPS` list will enable it.

### Test the changes we have made so far in our Django project

We can easily test our changes using the django development server.

Again, connect to the Vagrantbox, activate the virtual environment, then type:

`python manage.py runserver 0.0.0.0:8000`

`0.0.0.0` means make the server available on all network adapters on our development servers. `8000` is the port number. Rest is quite self-explanatory.

NOTE: If your django dev server keeps reloading, run the same command as above, but append the `--noreload` flag at the end.

Now, we can go to `127.0.0.1:8000` in our browser of choice, and verify that our project is up and running.
(We will see a rocket ship on the page)

### Django Models

In Django, we use models to describe the data we need for our application. Django then uses these models to setup and configure our database and store our data effectively. Each model in Django maps to a specific table in our database. Django handles the relationship between our models and tables for us so that we never have to write any SQL statements or interact with the database directly.

### Creating a Django Model

The first model we'll create is a user profile model. Out of the box, Django comes with a default user model that's used for the standard authentication system and the Django admin. We will override this model with our custom model to allow us to use an email address instead of a username that comes with the Django default model.

Best practice is to keep all of our models within the `models.py` file in our app in our project. In our case, that is the `models.py` file in the `profiles_api` app. Django has already imported `models` from `django.db` for us, but we'll also need some extra imports. These additional imports are:

`from django.contrib.auth.models import AbstractBaseUser`

`from django.contrib.auth.models import PermissionsMixin`

Above two imports are the standard base classes we need when overriding/customizing the default Django user model.

Models are described by making their classes. So, we need to create our `UserProfile` class and inherit from the above two imports.

```
class UserProfile(AbstractBaseUser, PermissionsMixin:
    """Database model for users in the system"""
```

After the above line, we should add a docstring that describes the purpose of the class. (Enclosed in triple double quotations)

Below the docstring, we can define the various fields our model should have.

```
email = models.EmailField(max_length=255, unique=True)
name = models.CharField(max_length=255)
is_active = models.BooleanField(default=True)
is_staff = models.BooleanField(default=False)
```

Fields definition is pretty self-explanatory.

Next, we need to specify the model manager that we're going to use for the model objects. This is required because we need to use our custom model with the CLI. So, Django needs to have a custom model manager for our custom user model so it knows how to create users and control users using the Django CLI.

`objects = UserProfileManager()`

We'll create the UserProfileManager class in a while.

We also need to add a couple more fields to allow our model to work with the Django admin and Django authentication system, because we're overriding the default model.

```
USERNAME_FIELD = 'email'
REQUIRED_FIELDS = ['name']

def get_full_name(self):
    """Retrieve full name of user"""
    return self.name

def get_short_name(self):
    """Retrieve short name of user"""
    return self.name

def __str__(self):
    """Return string representation of our user"""
    return self.email
```

First line tells Django to use the previously defined email field instead of the usernames Django uses by default. Second line defines the fields that should be marked required. (Since the `USERNAME_FIELD` (email) is required by default, we don't need to add it to the `REQUIRED_FIELDS` list). We have also added a couple of functions that help Django interact with our custom user model. They are quite self-explanatory. The last method `__str__` returns the string representation of our model. So, whenever some model object is printed, we'll get the email of that model as the result.

### Adding our User Model Manager

Now that our `UserProfile` model is created, we'll also create a manager for that model so that Django knows how to interact with it in the CLI.

One command we're going to be using in the CLI is called the `createsuperuser`. It allows us to create a user that has access to the Django admin page. But, since by default, Django expects a username and password, and our model is based on emails and passwords, we need to be able to tell Django to modify its default behaviour and look for emails instead of usernames. Our `UserProfileManager` will help us with that.

`UserProfileManager` will also be defined in the `models.py` file in our `profiles_api` app, but above our `UserProfile` model. Like `UserModel`, it will also inherit from a class called `BaseUserManager`. It can be imported as:

`from django.contrib.auth.models import BaseUserManager`

Our `UserProfileManager` class will be defined as:

```
class UserProfileManager(BaseUserManager):
    """Manager for user profiles"""
```

The way managers work is we specify some functions within the manager that can be used to manipulate objects of the model that the manager is for. So, the first function we'll need is the `create_user` function. This is the function the Django CLI will use when creating users.

```
def create_user(self, email, name, password=None):
    """Create a new user profile"""
    if not email:
        raise ValueError('User must have an email address')
    email = self.normalize_email(email)
    user = self.model(email=email, name=name)
    user.set_password(password)

    user.save(using=self._db)

    return user
```

`password` is set to `None` by default because of the way the Django password checking system works. A `None` password won't work because the password needs to be a hash. So, basically, until the user sets a password, he/she won't be able to authenticate his/her profile. Then, we'll check if the `email` is falsy. If it is, then we'll raise a `ValueError`. If the `email` is truthy, then we'll normalize it. Normalizing an email means setting the second half of that email (after @) to lowercase. Emails can be case sensitive in the first half, but not in the second half. Most email providers have everything case insensitive now, but still, it's best practice to normalize it. Next, we'll create our user model object. `self.model` is the model for which the manager is created. In our case, that is the `UserProfile` model. Note that we have only passed the `email` and `name` to the model, and not the `password`.
This is because passwords must be passed in another way, using the `set_password()` method provided by the `AbstractBaseUser`. This method encrypts the password, and we should always do that. Passwords should NEVER be stored as plaintext in our databases. After our model object has been fully defined, we can save it. The standard is to also specify the database we're saving to in the `user.save()` method since Django supports multiple databases. We're only going to be using one database, but it's still best practice to add the database in as a paramater in case we do decide to use another database. After creating the user model object, we can simply return it.

Next, we'll also create our `create_superuser` method:

```
def create_superuser(self, email, name, password):
    """Create and save a new superuser with given details"""
    superuser = self.create_user(email, name, password)
    superuser.is_superuser = True
    superuser.is_staff = True

    superuser.save(using=self._db)

    return superuser
```

Here, we haven't specified `password=None` because superusers must have a password. Rest is quite self-explanatory and similar to the `create_user`. One thing is that we hadn't defined the `is_superuser` field, so where's it coming from? It's coming from the `PermissionsMixin` that our `UserProfile` class inherits from.

### Setting our Custom User Model as the Default

Now that we have created our `UserProfile` model and its manager, we can configure our Django project to use this custom model and manager instead of the default.

For this, go to `settings.py` file in the `profiles_project` directory, and at the end, create a new line:

`AUTH_USER_MODEL = '<app_name>.<ModelName>'`

In our case, it would be:

`AUTH_USER_MODEL = 'profiles_api.UserProfile'`

### Creating Migrations and Syncing DB

Now, we'll create a migrations file for the models that we've added to our project. The way Django manages the database is it creates what's called a migration file that basically stores all of the steps required to make our database match our Django models. So, everytime we change a model, or add additional models to our project, we need to create a new migration file. It will contain the steps needed to match our database with our updated models.

We can create migrations using the Django CLI. Ensure that you're in the `/vagrant` directory and that the virtual environment is active, then type:

`python manage.py makemigrations <app_name>`

In our case, it would be:

`python manage.py makemigrations profiles_api`

Above command will create a new `migrations` directory containing our migration file in our project for our `profiles_api` app in the `profiles_api` directory.

We have done only one step so far, though. Now that our migrations file has been created, we also need to run it. For that, use:

`python manage.py migrate`

This will run all the migrations in our project.

This will create tables in the database corresponding to all of the models in our project and their dependencies.

### Creating a superuser

Django admin is a really useful tool that allows us to create an administrator website for our project through which we can manage our database models. This makes it really easy to inspect the database, and modify the models.

Before we can use the django admin, though, we'll have to create a superuser. superuser is a user with maximum privileges over our database. We'll create a superuser using the Django CLI. In the `/vagrant` directory with virtual environment active, type:

`python manage.py createsuperuser`

This will prompt you to enter your email, name, and password. Once entered, the superuser is created.

### Enabling Django Admin

By default, django admin is already enabled on all new projects. However, we need to register any newly created models with the django admin so that it knows that we want to display that model in the admin interface. To enable the django admin for our `UserProfile` model, go to the `admin.py` file in our `profiles_api` app directory. Import your model into this file by:

`from <app_name> import models`

In our case:

`from profiles_api import models`

After importing the model, register it using:

`admin.site.register(models.<ModelName>`

In our case:

`admin.site.register(models.UserProfile)`

admin is already imported by Django into the `admin.py` file.

### Testing Django Admin

Now that we have enabled the django admin for our project, we can test it by starting the django dev server using: (ensure virtual environment is active and you're in the `/vagrant` directory)

`python manage.py runserver 0.0.0.0:8000`

and going to:

`127.0.0.1:8000/admin`

Here, we'll type the email and password of the superuser we previously created.

We'll see three sections in the django admin interface. Each section represents a different app in our project. `AUTH TOKEN` app is automatically added as part of the `django rest framework`. `AUTHENTICATION AND AUTHORIZATION` is part of Django. It comes out of the box and allow us to use the authentication system. Finally, we have the app we created, `profiles_api`. In its section, we can also see the `UserProfile` model that we created. However, Django has automatically named it `user profiles`, by removing the pascal case and pluralizing it.

Clicking on the `user profiles` model, we can see our superuser.

### API Views

The Django Rest Framework offers a couple of helper classes we can use to create our API endpoints. A couple of them are:

1. `APIView`
2. `ViewSet`

Both classes are slightly different and offer their own benefits. We'll take a look at `APIView`.

`APIView` is the most basic type of view we can use to build our API. It enables us to describe the logic which makes our API endpoint. An `APIView` allows us to define functions that match the standard HTTP methods. (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`). By allowing us to customize the function for each HTTP method on our API URL, `APIView` gives us control over our API logic. This is great for implementing complex logic, e.g., calling other APIs, working with local files etc.

So, when should we use `APIView`? A lot of the times, it depends on preference. As we learn more about the Django Rest Framework. However, following are some situtations where it might be appropriate:

- Need full control over the logic
- Processing files and rendering a synchronous response
- Calling other APIs/services
- Accessing local files/data

### Creating our First APIView

Let's create a simple hello world APIView to understand how it works.

Go to the `views.py` file of the `profiles_api` app. Let's remove all the content of that file that is automatically inserted by Django, and add the following two imports:

```
from rest_framework.views import APIView
from rest_framework.response import Response
```

First line imports the `APIView`. Second imports the `Response`. This `Response` object is expected to return when django rest framework calls our `APIView`.

Let's define our `APIView`:

```
class HelloApiView(APIView):
    """Test API View"""
```

This creates a new class that inherits from `APIView`. It allows us to define the application logic for our endpoints. We will define a URL (our endpoint) and the django rest framework handles it by calling the appropriate function in the view for the HTTP request that is made.

The way `APIView` operates is that it expects a function for the different HTTP requests that can be made to the view. Let's define a function for the HTTP GET request.

```
def get(self, request, format=None):
    """Returns a list of APIView features"""
```

The `request` parameter is passed by the django rest framework and contains details of the request being made to the API. `format` is used to add a format suffix (e.g. `json`, `csv`, etc.) to the end of the endpoint URL. We won't be using formats, but it's still best practice to add the `format` parameter in case we decide to enable formats on our APIs.

```
an_apiview = [
    'Uses HTTP methods as functions (get, post, patch, put, delete)',
    'Is similar to a traditional Django View',
    'Gives you the most control over your application logic',
    'Is mapped manually to URLs',
]

return Response({
    'message': 'Hello',
    'an_apiview': an_apiview,
})
```

We have created a list that describes all the features of an `APIView` in our `get` function. Then, we return a `Response` object. It expects either a list or a dictionary, because `Response` object returns `json` data. For that, we need either a list or a dictionary.

### Configure View URL

Now that we have our `APIView`, we can wire this view up to a URL in Django. For that, go to the `urls.py` file in the `profiles_project` project folder. This is the entry point for all of the URLs in our app. This file already contains some data. We can see that the url to the `admin/` endpoint has already been added.

We need to create a `urls.py` file in our `profiles_api` app directory. This is where we'll store the URLs for our API. After creating this file, we need to add its entry in the `urlpatterns` list in the `urls.py` file in our `profiles_project` folder. For that, we first need to import `include` from `django.urls`. Since that module has already been imported, we simply need to append `include` at the end of that import, separated by a comma:

```
from django.contrib import admin
from django.urls import path, include
```

`include` is a function that we can use to include URLs from other apps in the root project `urls.py` file.

```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('<app_name>.urls'))
]
```

In our case, the `urlpatterns` list would look like:

```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('profiles_api.urls'))
]
```

Now, when we go to `/api` in the webserver, it will pass in the request to our django app (`profiles_api`), which will look for URL patterns for the first URL which matches the URL that we've entered. Anything after the `/api` in the URL that is provided in the browser is going to be passed to the `urls.py` file of the `profiles_api` app. We need to handle the rest of the URL in the `urls.py` file of `profiles_api` app:

```
from django.urls import path

from profiles_api import views

urlpatterns = [
    path('hello-view/', views.HelloApiView.as_view()),
]
```

This `urls.py` file is also quite similar to the `urls.py` file of the `profiles_project` directory. Here, we simply define the path, and the view we want to show for that path. In our case, we are going to show `HelloApiView` on `hello-view/` path. `as_view()` is the standard method we call to convert our `APIView` class to a form renderable in the browser. This view is now available on the URL `127.0.0.1:8000/api/hello-view`

### Create a Serializer

Serializer is a feature from the django rest framework
that allows us to easily convert data inputs into Python objects and vice versa. It's quite similar to a Django form which we define and it has various fields that we want to accept for the input for our api. For example, if we want to add `POST` or `UPDATE` functionality to our hello `APIView` then we need to create a serializer to receive the content that we post to the API.

To create a serializer, go to the `profiles_api` app directory, and create a `serializers.py` file.

```
from rest_framework import serializers


class HelloSerializer(serializers.Serializer):
    """Serializes a name field for testing our APIView"""
    name = serializers.CharField(max_length=10)
```

`name` is the field that we want to accept in our serializer input. `name` is a value that can be passed into the request that will be validated by the serializer.

Serializers also take care of validation rules. So, if we want to accept a certain field of a certain type, serializers will make sure that the content past that API is of the correct type.

### Add POST Method to APIView

In the `views.py` file in `profiles_api` app folder, include thefollowing two imports:

```
from rest_framework import status

from profiles_api import serializers
```

First import gives us access to the HTTP codes (200, 404 etc). Second is, of course, our `serializers` file.
We'll use our `serializers` file to tell our `APIView` what data to expect when making `POST`, `PUT`, and `PATCH` requests. We need to set a serializer in our `HelloApiView` to use the functionality of our serializer.

`serializer_class = serializers.HelloSerializer`

After adding the above line in our `HelloApiView`, we can now create our `post` method:

```
def post(self, request):
    """Create a hello message with our name"""
    serializer = self.serializer_class(data=request.data)

    if serializer.is_valid():
        name = serializer.validated_data.get('name')
        message = f'Hello, {name}!'
        return Response({'message': message})
    else:
        return Response(
            serializer.errors,
            status=status.HTTP_400_BAD_REQUEST,
        )
```

First, we retrieve the `serializer_class`. It is the standard way of retrieving `serializer_class` defined above when working with serializers in a view.

`data=request.data` assigns the data. When we make a `POST` request to our `APIView`, the data gets passed in as `request.data`. It's part of the `request` object that is passed to our `request` object. We pass this data to our serializer class and store the whole thing in a new local variable `serializer`. Next, we validate the serializer (ensure it is valid as per the specification of the serializer fields). If it is valid, then we're going to retrieve the valid data into the `name` variable (equivalent name to the field of the serializer). We'll construct a message using that field, and simply return our message as a `response`. If the data is not valid, then we'll simply return a 400 error. `serializer.errors` gives us a dictionary of all the errors that occurred based on the validation rules of the serializer. It's a good idea to return this, so that whoever faces the error has some idea of what went wrong. The second argument is the 400 bad request. By default, the `Response` object returns a 200, but, since we are facing an error, we'll send the 400 bad request error as the second argument.

We can send now send a `POST` request from `127.0.0.1:8000/api/hello-view`.

### Adding PUT, PATCH, and DELETE Methods in our APIView

We'll add dummy `PUT`, `PATCH`, and `DELETE` methods in our API:

```
def put(self, request, pk=None):
    """Handle updating an object"""
    return Response({'method': 'PUT'})

def put(self, request, pk=None):
    """Handle a partial update of an object"""
    return Response({'method': 'PATCH'})

def delete(self, request, pk=None):
    """Handle deleting an object"""
    return Response({'method': 'DELETE'})
```

All of these require a `pk` parameter since we must know which instance we are updating/deleting. Usually, the `pk` is the `id` of the object we want to manipulate.

All of these HTTP methods are now also available on `127.0.0.1:8000/api/hello-view`
