Key Commands:
$ docker container run -d --name nginx-test -p 8080:80 nginx
-d switch will run the container in detached mode
--name will provide name to the container nginx-test
$ docker container attach nginx-test 
Attach it will attach the running container to console
$ docker container attach --sig-proxy=false nginx-test
--sig-proxy=false CTRL+C won’t stop container, it will only return to console
You can use the exec
command; this spawns a second process within the container that you can interact with.
For example, to see the contents of the file /etc/debian_version, we can run the
following command:
$ docker container exec nginx-test cat /etc/debian_version

$ docker container exec -i -t nginx-test /bin/bash
This time, we are forking a bash process and using the -i and -t flags to keep open console
access to our container. The -i flag is shorthand for --interactive, which instructs
Docker to keep stdin open so that we can send commands to the process. The -t flag is
short for --tty and allocates a pseudo-TTY to the session.
While it is extremely useful as you can interact with the container as if it were a virtual
machine, I do not recommend making any changes to your containers as they are running
using the pseudo-TTY. It is more than likely that those changes will not persist and will be
lost when your container is removed.
$ docker container logs --tail 5 nginx-test
The logs commands is pretty self explanatory; it allows you to interact with the stdout
stream of your containers, which Docker is keeping track of in the background. For
example, to view the last entries written to stdout for our nginx-test container

To view the logs in real time,
$ docker container logs -f nginx-test
The -f flag is shorthand for --follow. I can also, say, view everything that has been
logged since 15:00 today by running the following:
$ docker container logs --since 2017-06-24T15:00 nginx-test

The logs command shows the timestamps of stdout recorded by Docker and not the time
within the container. You can see this from when I run the following commands:
$ date
$ docker container exec nginx-test date

There is an hour's time difference between my host machine and the container due to
British Summer Time (BST) being in effect on my host.
Luckily, to save confusion--or add to it, depending on how you look at it--you can add -t to
your logs commands:
$ docker container logs --since 2017-06-24T15:00 -t nginx-test

The -t flag is short for --timestamp; this option prepends the time the output was
captured by Docker:
The top command is quite a simple one; it lists the processes running within the container
you specify:
$ docker container top nginx-test
The stats command provides real-time information on either the specified container or, if
you don't pass a container name or ID, all running containers:
$ docker container stats nginx-test

Resource limits
Typically, we would have set the limits when we launched our container using the run
command; for example, to halve the CPU priority and set a memory limit of 128M, we
would have used the following command:
$ docker container run -d --name nginx-test --cpu-shares 512 --memory 128M
-p 8080:80 nginx

However, we didn't need to update our already running container; to do this, we can use
the update command. Now you must have thought that this should entail just running the
following command:
$ docker container update --cpu-shares 512 --memory 128M nginx-test


Running the preceding command will actually produce an error:
Error response from daemon: Cannot update container
e6ac3ce1418d520233a68f71a1e5cb38b1ab0aafab5fa00d6e0220975706b246: Memory
limit should be smaller than

So what is the memoryswap limit currently set to? To find this out, we can use the inspect
command to display all of the configuration data for our running container; just run the
following:
$ docker container inspect nginx-test
As you can see, there is a lot of configuration data. When I ran the command, a 199-line
JSON array was returned. Let's use the grep command to filter out just the lines that
contain the word memory:
$ docker container inspect nginx-test | grep -i memory
This returns the following configuration data:
"Memory": 0,
"KernelMemory": 0,
"MemoryReservation": 0,
"MemorySwap": 0,
"MemorySwappiness": -1,
Everything is set to 0: how can 128M be smaller than 0? In the context of the configuration
of the resources, 0 is actually the default value and means that there are no limits--notice the
lack of M at the end. This means that our update command should actually read the
following:
$ docker container update --cpu-shares 512 --memory 128M --memory-swap 256M
nginx-test

By default, when you set --memory as part of the run command, Docker will set the --
memory-swap size to be twice of that of --memory.
Also, rerunning docker container inspect nginx-test | grep -i memory will
show the changes:
"Memory": 134217728,
"KernelMemory": 0,
"MemoryReservation": 0,
"MemorySwap": 268435456,
"MemorySwappiness": -1,

Container states and miscellaneous commands
$ for i in {1..5}; do docker container run -d --name nginx$(printf "$i")
nginx; done

$ docker container pause nginx1
Running docker container ls will show that the container has a status of Up, but
Paused:
Note that we didn't have to use the -a flag to see information about the container as the
process has not been terminated; instead, it has been suspended using the cgroups freezer.
With the cgroups freezer, the process is unaware it has been suspended, meaning that it
can be resumed.
$ docker container unpause nginx1

Stop, start, restart, and kill
$ docker container stop nginx2
With this, a request is sent to the process for it to terminate, called a SIGTERM. If the process
has not terminated itself within a grace period, then a kill signal, called a SIGKILL, is sent.
This will immediately terminate the process, not giving it anytime to finish whatever is
causing the delay, for example, committing the results of a database query to disk.
Because this could be bad, Docker has given you the option of overriding the default grace
period, which is 10 seconds, by using the -t flag; this is short for --time. For example,
running the following command will wait up to 60 seconds before sending a SIGKILL,
should it need to be sent to kill the process:
$ docker container stop -t 60 nginx3
The start command, as we have already experienced, will start the process back up;
however, unlike the pause and unpause commands, the process in this case starts from
scratch rather than starting from where it left off.
$ docker container start nginx2 nginx3
The restart command is a combination of the following two commands; it stops and then
starts the container ID or name you pass it. Also, like stop, you can pass the -t flag:
$ docker container restart -t 60 nginx4
Finally, you also have the option sending a SIGKILL immediately to the container by
running the kill command:
$ docker container kill nginx5

Removing containers
To remove the two exited containers, I can simply run the prune command:
$ docker container prune
When doing so, a warning pops up and you are asked to confirm whether you are really
sure:
You can choose which container you want to remove using the rm command; like this, for
example:
$ docker container rm nginx4
However, attempting to remove a running container will result in an error:
Error response from daemon: You cannot remove a running container
5bbc78af29c871200539e7534725e12691aa7df6facd616e91acbe41f204b734. Stop the
container before attempting removal or use -f
As you can see from the preceding output, the error very kindly suggests that using the -f
flag will forcibly remove the container by stopping it and then removing it, requiring the
following command:
$ docker container rm -f nginx4
Another alternative would be to string the stop and rm commands together:
$ docker container stop nginx3 && docker container rm nginx3
Miscellaneous commands
The
create command is pretty similar to the run command, except that it does not start the
container, but instead prepares and configures one:
$ docker container create --name nginx-test -p 8080:80 nginx
You can check the status of your created container by running docker container ls -a
and then start the container with docker container run nginx-test before checking
the status again:
 
The next command we are going to quickly look at is the port command; this displays the
port along with any port mappings for the container:
$ docker container port nginx-test
It should return the following:
80/tcp -> 0.0.0.0:8080
diff command prints a list of all of the files that have been added or changed since the container started to run--so basically, a list of the differences on the filesystem between the image we used to
launch the container and now.
$ docker container diff nginx-test
This will return a list of files; as you can see from the following list, there is our testing file
along with the files that were created when nginx started:
C /run
A /run/nginx.pid
C /tmp
A /tmp/testing
C /var/cache/nginx
A /var/cache/nginx/client_temp
A /var/cache/nginx/fastcgi_temp
A /var/cache/nginx/proxy_temp
A /var/cache/nginx/scgi_temp
A /var/cache/nginx/uwsgi_temp


Docker networking
$ docker network create moby-counter
$ docker container run -d --name redis --network moby-counter redis:alpine
As you can see, we have used the --network flag to define which network our container
was launched in. Now that the Redis container is launched, we can launch the application
container by running the following:
$ docker container run -d --name moby-counter --network moby-counter -p 8080:80 russmckendrick/moby-counter
Again, we have launched the container into the moby-counter network; this time, we have
mapped port 8080 to port 80 on the container. Notice that we did not need to worry about
exposing any ports of the Redis container. That is because the Redis image comes with some
defaults, which expose the default port (which is 6379) for us. This can be seen by running
docker container ls
All that remains now is to access the application; to do this, open your browser and go to
http://localhost:8080/. You should be greeted by a mostly blank page with the
message Click to add logos:

Docker volumes
Check docker volume
$ docker volume ls
the volume name is not very friendly at all, in fact, it is the unique ID of the
volume. So how can we use the volume when we launch our Redis container? We know
from the Dockerfile that the volume was mounted at /data within the container, so all we
have to do is tell Docker which volume to use and where it should be mounted at runtime.
To do this, run the following command, making sure you replace the volume ID with that
of your own:
$ docker container run -d --name redis -v
719d0cc415dbc76fed5e9b8893e2cf547f0ac6c91233451604fdba31f0dd2d2a:/data --
network moby-counter redis:alpine
If your application page looks like it is still trying to reconnect to the Redis container once
you have launched your Redis container, then you may need to refresh your browser;
failing that, restarting the application container by running docker container restart
moby-counter and then refreshing your browser again should work.
You should be able to see the icons in their original positions. We can view the contents of
the /data folder on the Redis container by running:
$ docker container exec redis ls -lhat /data
This will return something that looks like the following:
total 12
drwxr-xr-x 1 root root 4.0K Jun 25 14:55 ..
drwxr-xr-x 2 redis redis 4.0K Jun 25 14:00 .
-rw-r--r-- 1 redis redis 421 Jun 25 14:00 dump.rdb
You can also remove your running container and relaunch it, but this time using the ID of
the second volume.
Finally, you can override the volume with your own. To create a volume, we need to use
the volume command:
$ docker volume create redis_data
Once created, we will be able to use the redis_data volume to store our Redis on by
running this command:
$ docker container run -d --name redis -v redis_data:/data --network mobycounter
redis:alpine
 
Like the network command, we can view more information on the volume using the
inspect command:
$ docker volume inspect redis_data
Look at the following output:
[
{
"Driver": "local",
"Labels": {},
"Mountpoint":
"/var/lib/docker/volumes/redis_data/_data",
"Name": "redis_data",
"Options": {},
"Scope": "local"
}
]
You can see that there is not much to a volume when using the local driver; one interesting
thing to note is that the path to where the data is stored on the Docker host machine is
/var/lib/docker/volumes/redis_data/_data. If you are using Docker for Mac or
Docker for Windows, then this path will be your Docker host VM and not your local
machine, meaning that you do not have direct access to the data inside the volume
DOCKER MACHINE
$ docker-machine ls
it lists the details on the machine name, the driver used and the Docker
endpoint URL, and the version of Docker the hosts are running
$ docker-machine ssh docker-local
command connects you to the Docker host(docker-local) using SSH:
$ docker-machine ip docker-local
There are also commands to stop, start,
restart, and remove your Docker host, though for now let's keep it running:
$ docker-machine stop docker-local
$ docker-machine start docker-local
$ docker-machine restart docker-local
$ docker-machine rm docker-local
There are also commands to find out more details about your Docker host:
$ docker-machine inspect docker-local
$ docker-machine config docker-local
$ docker-machine status docker-local
$ docker-machine url docker-local

$ docker-machine create \
--driver digitalocean \
--digitalocean-access-token
ba46b9f97d16edb5a1f145093b50d97e50665492e9101d909295fa8ec35f20a1 \
--digitalocean-image ubuntu-16-04-x64 \
--digitalocean-region nyc3 \
--digitalocean-size 512mb \
--digitalocean-ipv6 false \
--digitalocean-private-networking false \
--digitalocean-backups false \
--digitalocean-ssh-user root \
--digitalocean-ssh-port 22 \
docker-digitalocean
DOCKER-COMPOSE
$ docker-compose up -d
It will run in detach mode
$ docker-compose ps
When running these commands, Docker Compose will only be aware of the containers
defined in the service section of your docker-compose.yml file; all other containers will be
ignored as they don't belong to our service stack.
Config
Running the following command will validate our docker-compose.yml file:
$ docker-compose config
If there are no issues, it will print a rendered copy of your Docker Compose YAML file to
screen; this is how Docker Compose will interpret your file. If you don't want to see this
output and just want to check for errors, then you can run:
$ docker-compose config -q
The following command will read your Docker Compose YAML file and pull
any of the images it finds:
$ docker-compose pull
The following command will execute any build instructions it finds in your file:
$ docker-compose build
These commands are useful when you are first defining your Docker Compose-powered
application and want to test without launching your application. The docker-compose
build command can also be used to trigger a build if there are updates to any of the
Dockerfiles used to originally build your images.
The pull and build commands only generate/pull the images needed for our application;
they do not configure the containers themselves. For this, we need to use the following
command:
$ docker-compose create
This will create but not launch the containers. In the same way that the docker
container create command does, they will have an Exited state until you start them.
The create command has a few useful flags you can pass:
--force-recreate: This recreates the container even if there is no need to as
nothing within the configuration has changed
--no-recreate: This doesn't recreate a container if it already exists; this flag
cannot be used with the preceding flag
--no-build: This doesn't build the images, even if an image that needs to be
built is missing
--build: This builds the images before creating the containers
Start, stop, restart, pause, and unpause
The following commands work exactly in the same way as their docker container
counterparts, the only difference being they effect change on all of the containers:
$ docker-compose start
$ docker-compose stop
$ docker-compose restart
$ docker-compose pause
$ docker-compose unpause
It is possible to target a single service by passing its name; for example, to pause and
unpause the db service we would run:
$ docker-compose pause db
$ docker-compose unpause db
Top, logs, and events
$ docker-compose top
If you would like to see just one of the services, you simply have to pass its name when
running the command:
$ docker-compose top db
The next command streams the logs from each of the running containers to screen:
$ docker-compose logs
Like the docker container command, you can pass flags such as -f or --follow to keep
the stream flowing until you press Ctrl + C.
Also, you can stream the logs for a single service
by appending its name to the end of your command
Exec and run
$ docker-compose exec worker ping -c 3 db
This will launch a new process in the already running worker container and ping the db
container 3 times,
Scale
The scale command will take the service you pass the command and scale it to the number
you define; for example, to add more worker containers
$ docker-compose scale worker=3
However, this actually gives the following warning:
WARNING: The scale command is deprecated. Use the up command with the --
scale flag instead
What we should now be using is the following command:
$ docker-compose up -d --scale worker=3
While the scale command is in the current version of Docker Compose, it will be removed
from future versions


$ docker swarm <command>
$ docker node <command>
$ docker service <command>
$ docker stack <command>
