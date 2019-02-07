```
$ docker container [command]
$ docker network [command]
$ docker volume [command]
$ docker image [command]

docker build -t friendlyname .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname         # Same thing, but in detached mode
docker ps                                 # See a list of all running containers
docker stop <hash>                     # Gracefully stop the specified container
docker ps -a           # See a list of all containers, even the ones not running
docker kill <hash>                   # Force shutdown of the specified container
docker rm <hash>              # Remove the specified container from this machine
docker rm $(docker ps -a -q)           # Remove all containers from this machine
docker images -a                               # Show all images on this machine
docker rmi <imagename>            # Remove the specified image from this machine
docker rmi $(docker images -q)             # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry

docker stack ls              # List all running applications on this Docker host
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker stack services <appname>       # List the services associated with an app
docker stack ps <appname>   # List the running containers associated with an app
docker stack rm <appname>                             # Tear down an application
```
The following table summarizes the instructions; many of these options map 
directly to option in the "docker run" command:
Command	Description
ADD	Copies a file from the host system onto the container
CMD	The command that runs when the container starts
ENTRYPOINT	
ENV	Sets an environment variable in the new container
EXPOSE	Opens a port for linked containers
FROM	The base image to use in the build. This is mandatory and must be the first command in the file.
MAINTAINER	An optional value for the maintainer of the script
ONBUILD	A command that is triggered when the image in the Dcokerfile is used as a base for another image
RUN	Executes a command and save the result as a new layer
USER	Sets the default user within the container
VOLUME	Creates a shared volume that can be shared among containers or by the host machine
WORKDIR	Set the default working directory for the container

https://stackoverflow.com/questions/21553353/what-is-the-difference-between-cmd-and-entrypoint-in-a-dockerfile
The ENTRYPOINT specifies a command that will always be executed when the container starts.
The CMD specifies arguments that will be fed to the ENTRYPOINT.
If you want to make an image dedicated to a specific command you will use ENTRYPOINT ["/path/dedicated_command"]
Otherwise, if you want to make an image for general purpose, you can leave 
ENTRYPOINTunspecified and use CMD ["/path/dedicated_command"] as you will be able to override
the setting by supplying arguments to docker run.
For example, if your Dockerfile is:
```
FROM debian:wheezy
ENTRYPOINT ["/bin/ping"]
CMD ["localhost"]
Running the image without any argument will ping the localhost:
$ docker run -it test
PING localhost (127.0.0.1): 48 data bytes
56 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.096 ms
56 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.088 ms
56 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.088 ms
^C--- localhost ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.088/0.091/0.096/0.000 ms
Now, running the image with an argument will ping the argument:
$ docker run -it test google.com
PING google.com (173.194.45.70): 48 data bytes
56 bytes from 173.194.45.70: icmp_seq=0 ttl=55 time=32.583 ms
56 bytes from 173.194.45.70: icmp_seq=2 ttl=55 time=30.327 ms
56 bytes from 173.194.45.70: icmp_seq=4 ttl=55 time=46.379 ms
^C--- google.com ping statistics ---
5 packets transmitted, 3 packets received, 40% packet loss
round-trip min/avg/max/stddev = 30.327/36.430/46.379/7.095 ms
For comparison, if your Dockerfile is:
FROM debian:wheezy
CMD ["/bin/ping", "localhost"]
Running the image without any argument will ping the localhost:
$ docker run -it test
PING localhost (127.0.0.1): 48 data bytes
56 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.076 ms
56 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.087 ms
56 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.090 ms
^C--- localhost ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.076/0.084/0.090/0.000 ms
But running the image with an argument will run the argument:
docker run -it test bash
root@e8bb7249b843:/#
See this article from Brian DeHamer for even more details:
https://www.ctl.io/developers/blog/post/dockerfile-entrypoint-vs-cmd/

```

The ENTRYPOINT specifies a command that will always be executed when the container starts. 
The CMD specifies arguments that will be fed to the ENTRYPOINT. is a good to-the-point summary

Sometimes you see COPY or ADD being used in a Dockerfile, but 99% of the 
time you should be using COPY, here's why: COPY and ADD are both Dockerfile 
instructions that serve similar purposes. They let you copy files from 
a specific location into a Docker image. COPY takes in a src and destination
. It only lets you copy in a local file or directory from your host (the 
machine building the Docker image) into the Docker image itself. ADD lets 
you do that too, but it also supports 2 other sources. First, you can use
a URL instead of a local file / directory. Secondly, you can extract a 
tar file from the source directly into the destination.In most cases if 
you’re using a URL, you’re downloading a zip file and are then using the
RUNcommand to extract it. However, you might as well just use RUN with 
curl instead of ADDhere so you chain everything into 1 RUN command to
make a smaller Docker image.A valid use case for ADD is when you want 
to extract a local tar file into a specific directory in your Docker image. 
This is exactly what the Alpine image does with ADD rootfs.tar.gz /.
If you’re copying in local files to your Docker image, always 
use COPY because it’s more explicit.


