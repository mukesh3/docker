Get Docker CE on CentOS
* Set up the repository
Set up the Docker CE repository on CentOS:
```
$sudo yum install -y yum-utils
$sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
$sudo yum makecache fast
```
* Get Docker CE
Install the latest version of Docker CE on CentOS:</br>
```sudo yum -y install docker-ce```

* Start Docker:</br>
```$sudo systemctl start docker```
* Test your Docker CE installation
Test your installation:</br>
```$sudo docker run hello-world```

Uninstall Docker CE:
* Uninstall the Docker package:
```
$ sudo dnf remove docker-ce
```
* Images, containers, volumes, or customized configuration files on your host are not automatically removed. 
To delete all images, containers, and volumes:
```
$ sudo rm -rf /var/lib/docker
```
Complete instruction at https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/centos/

* Post-installation steps for Linux:
Manage Docker as a non-root user: The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket
is owned by the user root and other users can only access it using sudo. The docker daemon 
always runs as the root user.If you don’t want to use sudo when you use the docker command,
create a Unix group called docker and add users to it. When the docker daemon starts, 
it makes the ownership of the Unix socket read/writable by the docker group.
Warning: The docker group grants privileges equivalent to the root user. 
For details on how this impacts security in your system, see Docker Daemon Attack Surface.
To create the docker group and add your user:
Create the docker group.
```
$ sudo groupadd docker
Add your user to the docker group:
$ sudo usermod -aG docker $USER
```
Log out and log back in so that your group membership is re-evaluated.
If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take affect.On a desktop Linux environment such as X Windows, log out of your session completely and then log back in.
Verify that you can run docker commands without sudo:
```
$ docker run hello-world
```
Configure Docker to start on boot:
Systemd:
```
$ sudo systemctl enable docker
To disable this behavior, use disable instead.
$ sudo systemctl disable docker
```
