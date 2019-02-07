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

Complete instruction at https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/centos/
