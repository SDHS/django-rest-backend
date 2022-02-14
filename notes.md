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
