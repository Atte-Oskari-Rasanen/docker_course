# docker_course
Notes and exercises from docker course from helsinki university

Docker course
https://devopswithdocker.com/getting-started


#quick commands:
Remove ALL containers
docker rm -vf $(docker ps -aq)
Remove ALL images
docker rmi -f $(docker images -aq)

Before you can remove an image you must remove the container using:
docker container rm <cont>

then remove image
docker image rm <im>

Notice that the command also works with the first few characters of an ID.
For example, if a container's ID is 3d4bab29dd67, you can use docker container
rm 3d to delete it. Using the shorthand for the ID will not delete multiple containers,
so if you have two IDs starting with 3d, a warning will be printed, and neither will
be deleted. You can also use multiple arguments: docker container rm id1 id2 id3

You can also use the image pull command to download images without running them: docker image pull hello-world

Run a container:

$ docker container run nginx

docker container run -d nginx  #runs the container in the background ("detached")

YOU CANNOT REMOVE A RUNNING CONTAINER, need to stop it first:
docker container stop <container>

Exercise 1.1 output
(base) atte@atter-ThinkPad-X240:~/Documents/Devops$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                       PORTS     NAMES
2b98bb3f7412   nginx     "/docker-entrypoint.…"   57 seconds ago       Exited (0) 19 seconds ago              wonderful_khayyam
d3ce6fd7900a   nginx     "/docker-entrypoint.…"   About a minute ago   Exited (0) 9 seconds ago               sharp_ptolemy
062e9a51b68c   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute            80/tcp    interesting_dhawan

Exercise 1.2 output
(base) atte@atter-ThinkPad-X240:~/Documents/Devops$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
062e9a51b68c   nginx     "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   80/tcp    interesting_dhawan

(base) atte@atter-ThinkPad-X240:~/Documents/Devops$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    c919045c4c2b   2 weeks ago   142MB


#more commands
-i (interactive), -t (tty) and -d (detached)
$ docker run -d -it --name looper ubuntu sh -c 'while true; do date; sleep 1; done'

#so we are running an interactive ubuntu OS (image name we use is ubuntu) which can take in commands and detached
#Because we ran the container with --name looper, we can now reference it easily.
#To check that it is running:
docker logs -f looper
#we can pause the looper via
docker pause looper
#Keep the logs open and attach to the running container from the second terminal using 'attach':
$ docker attach looper

Let's start another process with -it and add --rm in order to remove it automatically
after it has exited. The --rm ensures that there are no garbage containers left behind.
It also means that docker start can not be used to start the container after it has exited.
$ docker run -d --rm -it --name looper-it ubuntu sh -c 'while true; do date; sleep 1; done'

Exec 1.2
docker exec -it elastic_ride bash
root@b59e744b329f:/usr/src/app# tail -f ./text.log
Secret message is: 'You can find the source code here: https://github.com/docker-hy'
https://github.com/docker-hy'


To get a list of existing containers and their names simply invoke
>docker ps -a

##########
made a following edit. maybe comment the line later
sudo vim /etc/gai.conf
change line ~54 to uncomment the following:

precedence ::ffff:0:0/96  100
###########


Exec 1.3
docker run -d -it ubuntu sh -c -it -d 'echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;'

Images are composed of different layers that are downloaded in parallel to speed up the download

We can also tag images locally for convenience, for example, docker tag ubuntu:18.04 ubuntu:bionic creates the tag ubuntu:bionic which refers to ubuntu:18.04.

Tagging is also a way to "rename" images. Run docker tag ubuntu:18.04 fav_distro:bionic and check docker images to see what effects the command had.


Exec 1.5
REPOSITORY                          TAG       IMAGE ID       CREATED         SIZE
devopsdockeruh/simple-web-service   ubuntu    4e3362e907d5   12 months ago   83MB
devopsdockeruh/simple-web-service   alpine    fd312adc88e0   12 months ago   15.7MB

/usr/src/app # tail -f ./text.log
2022-03-19 23:33:08 +0000 UTC
Secret message is: 'You can find the source code here: https://github.com/docker-hy'


Sample Dockerfile
#####
# Start from the alpine image that is smaller but no fancy tools
FROM alpine:3.13

# Use /usr/src/app as our workdir. The following instructions will be executed in this location.
WORKDIR /usr/src/app

# Copy the hello.sh file from this location to /usr/src/app/ creating /usr/src/app/hello.sh
COPY hello.sh .

# Alternatively, if we skipped chmod earlier, we can add execution permissions during the build.
# RUN chmod +x hello.sh

# When running docker run the command will be ./hello.sh
CMD ./hello.sh
#####

#Then build the image (when in the dir where the file is)
docker build . -t hello-docker
The intermediate containers are containers created from the image in which the command
is executed. Then the changed state is stored into an image

you can e.g. copy new files into the running container via
docker cp ./additional.txt zen_rosalind:/usr/src/app/

and check that the changes have been made
docker diff <container_name>

e.g.
2 $ docker diff zen_rosalind
    C /usr
    C /usr/src
    C /usr/src/app
    A /usr/src/app/additional.txt
    C /root
    A /root/.ash_history
The character in front of the file name indicates the type of the change in the
container's filesystem: A = added, D = deleted, C = changed.

Next we will save the changes as a new layer!

2 $ docker commit zen_rosalind hello-docker-additional
    sha256:2f63baa355ce5976bf89fe6000b92717f25dd91172aed716208e784315bfc4fd
2 $ docker images
    REPOSITORY                   TAG          IMAGE ID       CREATED          SIZE
    hello-docker-additional      latest       2f63baa355ce   3 seconds ago    5.57MB
    hello-docker                 latest       444f21cf7bd5   31 minutes ago   5.57MB

#V2 Dockerfile
############
# Start from the alpine image
FROM alpine:3.13

# Use /usr/src/app as our workdir. The following instructions will be executed in this location.
WORKDIR /usr/src/app

# Copy the hello.sh file from this location to /usr/src/app/ creating /usr/src/app/hello.sh.
COPY hello.sh .

# Execute a command with `/bin/sh -c` prefix.
RUN touch additional.txt

# When running docker run the command will be ./hello.sh
CMD ./hello.sh
##############


#lets build it
docker build . -t hello-again:v2
#and check its files
docker run hello-again:v2 lsadditional.txt
hello.sh

So instructions in a Dockerfile except CMD are executed during build time.
CMD is executed when we call docker run, unless we overwrite it.




Exec 1.6
#Create a two line dockerfile, use FROM and CMD to create a brand new image that automatically runs the server

Dockerfile:
############
FROM devopsdockeruh/simple-web-service:alpine
CMD server
############

(base) atte@atter-ThinkPad-X240:~/Documents/Devops/webserver$ docker build . -t web-server
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM devopsdockeruh/simple-web-service:alpine
 ---> fd312adc88e0
Step 2/2 : CMD server
 ---> Running in 3270b0100430
Removing intermediate container 3270b0100430
 ---> 1df57a3bb0a4
Successfully built 1df57a3bb0a4
Successfully tagged web-server:latest
(base) atte@atter-ThinkPad-X240:~/Documents/Devops/webserver$ docker run web-server
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /*path                    --> server.Start.func1 (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080


#Entrypoint used instead of CMD since if we run an image requiring and URL with CMD in the
Dockerfile, then passing the URL as a parameter when running "docker run" wont work!

'''
FROM ubuntu:18.04

WORKDIR /mydir

RUN apt-get update && apt-get install -y curl python
RUN curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
RUN chmod a+x /usr/local/bin/youtube-dl

ENV LC_ALL=C.UTF-8

# Replacing CMD with ENTRYPOINT
ENTRYPOINT ["/usr/local/bin/youtube-dl"]
'''
$ docker build -t youtube-dl .
$ docker run youtube-dl https://imgur.com/JY5tHqr

ENTRYPOINT vs CMD can be confusing - in a properly set up image such as our youtube-dl
the command represents an argument list for the entrypoint. By default entrypoint is
set as /bin/sh and this is passed if no entrypoint is set. This is why giving path
to a script file as CMD works: you're giving the file as a parameter to /bin/sh.

In addition, there are two ways to set them: exec form and shell form. We've been
using the exec form where the command itself is executed. In shell form the command
that is executed is wrapped with /bin/sh -c - it's useful when you need to evaluate
environment variables in the command like $MYSQL_PASSWORD or similar.



