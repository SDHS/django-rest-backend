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
|_init.py
|_settings.py
|_urls.py
|_wsgi.py
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
