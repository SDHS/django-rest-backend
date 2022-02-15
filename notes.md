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

First line tells Django to use the previously defined email field instead of the usernames Django uses by default. Second line defines the fields that should be marked required. (Since the `USERNAME_FIELD` (email) is required by default, we don't need to add it to the `REQUIRED_FIELDS` list). We have also added a couple of functions that help Django interact with our custom user model. They are quite self-explanatory. The last method `__self__` returns the string representation of our model. So, whenever some model object is printed, we'll get the email of that model as the result.

### Adding our User Model Manager

Now that our `UserProfile` model is created, we'll also create a manager for that model so that Django knows how to interact with it in the CLI.

One command we're going to be using the CLI is called the `create_superuser`. It allows us to create a user that has access to the Django admin page. But, since by default, Django expects a username and password, and our model is based on emails and passwords, we need to be able to tell Django to modify its default behaviour and look for emails instead of usernames. Our `UserProfileManager` will help us with that.

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

    user.save(user=self._db)

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

    superuser.save(self._db)

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
