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
