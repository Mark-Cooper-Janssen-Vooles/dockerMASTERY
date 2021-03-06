# Docker

#### Why docker, why now?

- Released in 2013 
- Recap:
  - mainframe to pc (90s)
  - baremetal to virtual (00's)
  - datacenter to cloud (10's)
  - host to container (serverless, 20's)
- Docker is all about speed => gets things done faster
  - develop, build, test, deploy, update, recover faster!
- Containers reduce complexity, its consistent accross the board regardless of your OS etc.
- 80% of time is spend managing existing software, only 20% on innovation, docker reduces maintence time and allows us to innovate more


## Section 3: Creating and using containers
``docker version`` => verify cli can talk to engine

*Images vs containers*
- An image is the application we want to run
- A container is an instance of that image running as a process
- You can have many containers running off the same image
- Dockers default image registary is called "docker hub" (hub.docker.com)

*Run / stop / remove containers*

- ``docker container run --publish 80:80 nginx`` (old way was "docker run") => starts a new container from an image. Can now go to "localhost" in the browser"
- With above command, docker looked for an image called "nginx" from docker hub.
- Found it, pulled down lates image and started a new container from that image 
- publish opens post 80 on the host ip
- routes that traffic to the contaier IP, port 80 (since nginx is a web server, it defaults to port 80)
- You'll get a 'bind' error if the left number (host post) is being used by anything else. You can use any port you want on the left, like 8888:80, then use localhost:8888 when testing
- ctrl + c sends a stop stop signal to the process when running in foreground (terminal)
- ``docker container run --publish 80:80 --detach nginx`` leaves container to run in the background, and we get back the unique container id.
- to list current RUNNING containers ``docker container ls`` (docker ps was the old way)
- to stop a container: ``docker container stop <containerid>`` note: for the container id, you just have to type the first few digits (enough for it to be unique)
- ``docker continer ls -a`` will list all the containers that have been created. 
- Note: every time you do a ``docker container run`` command, it starts a new container. 
- Note: the container id has to be unique, and is always created for us. The container name is also created for us if we dont specify it. i.e. ``docker container run --publish 80:80 --detach --name webhost nginx`` to specify the name of 'webhost'
- ``docker container logs webhost`` will spit out the latest logs from the webhost
- ``docker container top webhost`` => this is the process running inside the container
- ``docker container ls -a`` shows all 3 containers. To remove multiple containers, can run ``docker container rm 63f 699 0de <as many container ids as you want>`` (old way was ``docker rm`` ) => it won't let you remove running containers, unless you do ``docker container rm -f 63f``
- ``docker image ls`` shows all the images you have

**What happens in 'docker container run**
1. Looks for image (i.e nginx) locally in image cache, doesn't find anything
2. Then looks in remote image repository (defaults to docker hub)
3. Downloads the latest version (nginx:latest by default)
4. Creates new container based on that image and prepares to start 
5. Gives it a virtual IP on a private network inside docker engine 
6. 80:80 Opens up port 80 on host and forwards to port 80 in container. 
7. Starts container by using the CMD in the image dockerfile

Example of changing the defaults:
``docker container run --publish 8080:80 -name webhost -d nginx:1.11 nginx -T``

**Containers aren't mini VM's**
- They are just processes
- Limited to what resources they can access
- Exit when process stops

``docker run -name mongoDB -d mongo``
``docker ps`` => confirm its running
``docker top mongoDB`` => lists running processes in specific container
``ps aux`` "show me all running processes"

assignment: manage multiple containers
docs.docker.com and --help are your friend
* recommends always using --detch or -d, and name them with --name
* nginx should listen on 80:80, httpd on 8080:80, mysql on 3306:3306
* when running mysql use the --env option (or -e) to pass in MYSQL_RANDOM_ROOT_PASSWORD=yes
* use docker container logs on mysql to find the random password created on startup
* clean it all up with docker container stop and docker container rm (both can accept multiple names or IDs)
* use docker container ls to ensure everything is correct before and after cleanup

``docker container run --publish 80:80 --detach --name nginxMMC nginx``

``docker container run --publish 8080:80 --detach -e MYSQL_RANDOM_ROOT_PASSWORD=yes --name mysqMMC``  mysql (order was important, needed -e before --name)
his looked like => ``docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql``

``docker container run --publish 3306:3306 --detach --name httpdMMC httpd``


``docker container logs mysqMMC``

can do "curl localhost" to get nginx (80:80), and "curl localhost:8080" to get apache 

``docker container stop nginxMMC mysqMMC httpdMMC``

``docker container rm nginxMMC mysqMMC httpdMMC``

**Whats going on in containers**
``docker container top <name or id>`` - process list in one container

``docker container inspect <name or id>`` - details of one containger config. Gives back a json array of metadata about container (startup config, volumes, networking, etc)

``docker container stats`` - performance stats for all containers. Gives a live stream view of container information about cpu usage etc. Unfortunately only gives container id and not name 

**Getting a shell in containers**
=> getting into a container to mess around with the inside
No need for SSH 
``docker container run -it`` - start a new container interactively
``docker container exec -it`` - run additional command in existing container 
- different linux distros inside 

``docker container run -it --name proxy nginx bash`` => you're in a shell/terminal!! you an go "ls" and see all the files etc in the root of the image. You can download stuff etc, do just about any common thing. type "exit" to leave the shell.
Note: The thing about ubuntu and other distributions inside a container is that they;re usually very minimal. A live  CD / download of the ISO of ubuntu, which you would normally put on a virtual machine,  is going to have a lot more software installed by default compared to a ubuntu container (which is a very minimal version by default - but you can always add more to it)

If you want to start a container (instead of run which creates a new one) in terminal mode, ``docker container start -ai <name of container or id>``

To see the shell inside an already running container: 
``docker container exec -it <name of container or id> bash``
=> when you exit the bash shell with ``exit``, ``docker ps`` shows the container is still running because exec actually runs an additional process on an existing running container, it wont effect the root process. 

"Alpine" is a small security-focused distribution of linux, only 5mg in size!
Alpine comes with its own package manager

If we go ``docker container run -it alpine bash`` it says "docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"bash\": executable file not found in $PATH": unknown."
=> Basically, we can only run things in the container that already exist in its image when you started it. Bash doesn't exist in alpine yet ... so how to get in its image? 
=> Alpine is so small it doesn't have bash, but it does have sh. so you can run ``docker container run -it alpine sh`` and if you search alpine on the internet, its package manager is apk and we could use this to install bash if we needed it. 

**Docker Networks: Concepts**
- ``docker container run -p`` => exposes the port on your machine. defaults normally "just work"
- quick port check with ``docker container port <container>``


docker network defaults: 
- When you start a container, in the background you're really connecting to the docker network (usually the docker bridge network)
- Each virtual network routes through NAT firewall on host IP
- All containers on a virtual network can talk to each other without -p
- Docker saying "batteries included, but removable" means that defaults normally work, but you can tweak them.
- You can make new virtual networks 
- You can attach containers to more than one virtual network (or none)
- Skip virtual networks and use host IP (--net=host)

---

Command line stuff:
- ``docker container run -p 80:80 --name webhost -d nginx`` => 80:80 is always host:container format. I.e. 80 is the default and represents "https://localhost". If you use 8080:80, your container will be on https://localhost:8080
- ``docker container port webhost`` shows us which ports are forwarding traffic from the host to that container itself
- ``docker container inspect --format '{{.NetworkSettings.IPAddress}} webhost' `` => --format, a common option for formatting the output of commands using "Go templates", you can see the address of the container is not on the same IP network as the host computer 

---

Docker Networks: CLI Management 

nginx:alpine

- Show networks ``docker network ls``
  - "bridge" is the default docker virtual network, that bridges through the NAT firewall, to the physical network that your host is connected to
- Inspect a network ``docker network inspect <name of network>``
- Create a network ``docker network create --driver``
  - If you don't put --driver in there, it will default to bridge
- Attach a network to container ``docker network connect``
- Detach a network from container ``docker network disconnect``
- To attach a network to a container, can attach it i.e. when you create a network: ``docker container run -d --name new_ngnix --network my_app_net nginx``
Another way to connect to a network: 
``docker network --help``
``docker network connect <put the network id here> <put the app network here>``
If you connect a container to another host, it will now be on two networks!
You can check it has connected to the new network with: ``docker container inspect <container id>``
- to disconnect a container from a network: 
``docker disconnect <network id> <container id>``

If you're running containers on a single server, you can protect them by giving them their own network. 
Create your apps so frontend/backend sit on the same Docker network.
Their inter-communication never leaves host
All externally ports closed by default, you must manually expose via -p 
This gets even better later with Swarm and Overlay networks 

===

Docker Networks: DNS

- Understand how DNS is the key to easy inter-container comms
  - Forget IP's , static IP's and using IP's for talking to containers is best avoided
  - A network which does not have the default 'bridge' gets a special new feature: automatic DNS resolution
  - If you create another container on that network, it will be able to connect to all existing containers on that network, 
  ``docker container exec -it <first container> ping <second container>`` should connect.
  - Note: the default bridge network does not have the DNS server built into it by default. You can use ``--link`` to create manual links between containers, but it much easier to create a new network for your apps. 
- See how it works by default with custom networks
- Learn how to use --link to enable DNS on default bridge network

Key take aways: 
Containers shouldn't rely in IP's for inter-communication, DNS for friendly names is built-in if you use custom networks. 

===

ASSIGNMENT: Using Containers for CLI testing
``docker container run -it centos:7 bash``
``docker container run -it ubuntu:14.04 bash``
``docker container rm 789 987`` => cleanup!


if you use:
``docker container run --rm -it ubuntu:14.04 bash`` the --rm basically removes the container once you edit out of the terminal

===

Assignment: DNS Round robin test 
you can have two different hosts, with DNS aliases, that respond to the same DNS name.
i.e. google.com has more than 1 server

Since docker engine 1.11, we can have multiple containers on a created network respond to the same DNS address
Step 1: 
Create network
``docker network create dude``
Step 2: 
Create two containers
``docker container run -d --net dude --net-alias search elasticsearch:2``
Step 3:
Run test to make sure you can get to both with same DNS name
``docker container run --rm --net dude alpine:3.10 nslookup search``
it should return 2 values
Step 4: 
Run test with centos 
``docker container run --rm --net dude centos curl -s search:9200``

====

## Docker Images
- the building blocks of containers
- Whats in an image (and what isnt)
- Using docker hub registry
- Managing our local image cache
- Building our own images

***Whats in an image (and what isnt)***
- App binaries and dependencies
- Metadata about the image data and how to run the image
- Official definition: "An image is an ordered collection of filesystem changes and the corresponding execution parameters for use within a container runtime"
- Not a complete OS (no kernel or kernel modules. just the binaries the container needs, as the host provides the kernel)
=> More like starting an application than booting up a whole operating system (which is what virtual machine would do)

---

***Using dockerhub registery images***
- basics of docker hub (hub.docker.com)
- find offocial and other good public images
  - good vs bad images
- download images and basics of image tags

When on the website, an official image will have no "/" in it! When a normal person makes an image, it will have our account name infront of it like: "march4/myimage"

Official images:
- docker has a team of people who take care of it, ensure docs are updated etc.
- When you start you'll want to just use official images. Great docs on how to make them work, options, env variables, default ports etc.
- images aren't necessarily tagged, they're named
- to download an image: 
``docker pull <image name>`` => will automatically download the latest one
``docker pull nginx:1.18.0`` will download that specific version
- at this link: https://hub.docker.com/_/nginx the "tags" are interchangable, i.e. this would be the same as the above command: ``docker pull nginx:stable``
- in software, its best to specify the exact option, don't want it to automatically update!
- dockerhub has lots of open source images created by users like you and I. Very similar to github. If you ever want to use a non-official image, look for number of stars and number of pulls as a popular repository tends to establish trust!

***The image cache**
- image layers
- union file system
- history and inspect commands
- copy on write

image layers: 
``docker image ls`` shows the images 
``docker image history <image name>`` shows the history of the image layers. Every image starts from a blank layer (scratch), every set of changes that happens after that is another layer. 
=> We don't need to download layers we already have! I.e. If we have a custom image, but they both use the same base ubuntu layer, that layer will be used for both images (saves time and space on the host)
- Note: the <missing> tag refers to them just being layers inside the image. Theyre not images themselves, so they dont get their own image ID.
- ``docker image inspect <image name>`` => this gives us all the JSON metadata about the image

**Image tagging and pushing to dockerhub**

- ``docker image tag --help``, images don't technically have a name so we have to refer to them by three pieces of info: <user>/<repo>:<tag> (default tag is latest if not specified)
if we go ``docker image ls``, we're probably only dealing with official images so we're not going to see the user repository name. Offical repositiroes live at the root namespace, so they dont need an account name infront of the repo name.
- the tag is not quote a version or a branch, similar to git tags. Its a pointer to a specific image commit
- You can retag existing docker images: ``docker image tag <imagine youre going to give a new tag to> <the new tag>``
- i.e. ``docker image tag mongo march4/mongo`` will make a new repo with the same image id as the existing mong
- to push it: ``docker image push march4/mongo``. it will deny unless you're logged in 
- to login "docker login": NOTE: Whenever you log on to a computer with your docker info, it will store an authentication key for your profile for that user. If you're using a machine you dont trust, type "docker logout" 
- now push image again. Go to dockerhub and hit refresh!
- to add a tag: ``docker image tag march4/mongo march4/mongo:1.4.1``, ``docker image push march4/mongo:1.4.1``
- to make a private repo, create the repository first in the dockerhub web UI. Then push to it, and it will never be on the internet for others.

***The dockerfile basics***
A recipe for creating your image!

look in udemy-docker-mastery/dockerfile-sample-1

looks similar to a shell script, but its not!

The "stanza's" (or commands) run top-down, so order is important. 

- FROM stanza => in every docker file, it needs to be there. normally a minimum distribution, like alpine. 

Package managers like apt and yum are one of the reasons to build containers FROM Debian, Ubuntu, Fedora or CentOS

- ENV => for environment variables, a way to set them. Theyre the main way we set keys and values for container building and running containers. Any subsequent lines will be able to use this.

- RUN => essentially its executing shell commands. It uses && to chain commands one after the other. Each stanza is one layer, so this chaining with the && makes sure all these commands are fit into one layer

The next RUN command is linking standard in and standard out logs 

- EXPOSE => by default no ports are opened by a container: it doesn't expose anything from a virtual container unless we list it here

- CMD => required paramater. Final command that will be run every time you launch a new container from the image, or every time you restart a stopped container. 

--- 

***Building Images: Running docker builds***

``docker image build -t customnginx .``
the -t adds the tag and the . means build this image in this directory. 

If you change a Dockerfile and rebuild, it will use the cached stuff if it hasn't changed. 

Its importortant to keep the stuff at the top that wont change much, and the stuff that might change more at the bottom of the dockerfile to save time!

**Building images: Extending existing images**

(in dockerfile-sample-2)

=> In simple scenarios, the offical images work.
=> As our projects grow in complexity etc, we'll need to add more config 

``WORKDIR /usr/share/nginx/html`` Changes working directory to root of nginx webhost 
=> using WORKDIR is preferred to using ``RUN cd /some/path``, but would do the same thing!

=> ``COPY index.html index.html``: This is the stanza you use to copy your source code from your local machine into your container images (in this instance it takes the index.html and puts it in the nginx webhost directory that we WORKDIR'd into above)

=> Note: We're not specifying a CMD in this dockerfile, how to we get away with this? 
- Theres already a CMD specified in the FROM image. When we use the image, we're inheriting everything from the Dockerfile used to generate the image that we're FROMing
- Thats how you can chain dockerfiles together!

``docker container run -p 80:80 --rm nginx``
=> use above to see default nginx server html by visiting "localhost"

now run the dockerfile:
``docker image build -t nginx-with-html .``
now if we go into our ``docker image ls`` we can see a new image called "nginx-with-html"
so we repeat the command but change the image:
``docker container run -p 80:80 --rm nginx-with-html``
=> use above to see NEW nginx server html by visiting "localhost" (need to do a hard refresh)

To send this up to our docker repo we'd have to tag it by adding our account to the front i.e: 
``docker image tag nginx-with-html:latest march4/nginx-with-html:latest``
``docker image push march4/nginx-with-html``


===

Assignment: Build your own image 
Step 1 - Write Dockerfile:
````dockerfile
FROM node:6.10-alpine
EXPOSE 3000 # this is the port you want to expose from inside the container (this node app runs on localhost:3000)
RUN apk add --update tini \
    && mkdir -p /usr/src/app 
WORKDIR /usr/src/app
COPY package.json package.json
RUN npm install \
    && npm cache clean --force 
COPY . . 
CMD /sbin/tini -- node ./bin/www
````

Step 2 - Build image: 
``docker image build -t march4/dockerfile-assignment-1 .``
Confirm image is built ``docker image ls``

Step 3 - Start container: 
``docker container run -p 80:3000 -rm march4/dockerfile-assignment-1``

Step 4 - Put on dockerhub: 
``docker image push march4/dockerfile-assignment-1``

***Using Prune to keep your docker system clean***

- ``docker image prune`` to clean up just 'dangling' images
- ``docker system prune`` will clean up everything
- most common is ``docker image prune -a`` which removes all images you're not currently using. 
- ``docker system df`` to see space usage

===

## Section 5: Container Lifetime and Persistent Data - volumes, volumes, volumes

- Defining the problem of persistent data
- Key concepts with containers: immutable, ephemeral
- Learning and using Data Volumes
- Learning and using Bind Mounts
- Preserve database data while replacing containers for databases (VOLUME)
- mounting code into a container from the host while you're editing it so you can run that code in the host while youre editing it live (BIND MOUNT)


**Container lifetime & Persistent data**
- Containers are usually immutable and ephemeral
- If we change something, we just redeploy the whole container 
- This is the ideal scenario, but what bout databases or unique data? Docker gives us features to ensure these 'separation of concerns'
- This is known as 'persistent data'
- Two ways: Volumes and Bind Mounts
- Volumes: make special location outside of container UFS 
- Bind mounts: Link container path to host path

***Persistent Data: Volumes***
- Can use ``VOLUME`` command in Dockerfile
- i.e. take a look at a dockerfile: https://github.com/docker-library/mysql/blob/fef511444a9d2867c9e4e20f5b4062bc071c20a2/8.0/Dockerfile
On line 77 it says where the volume is stored
- NOTE: Volumes need manual deletion, you can't clean them up just by deleting a container!
- Can do ``docker volume ls`` to see a list of volumes
- ``docker volume inspect <volume id here>`` 
- issue with ``docker volume ls`` => very hard to know what volume belongs to what. If you have multiple containers and delete them, the volumes will not be deleted. But what containers did the volumes belong to if you want to attach them to a new container?
- ``docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v /var/lib/mysql mysql`` => this is the default path
- ``docker container run -d --name mysqlVolume -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql`` the ``mysql-db:/var...`` is know as a 'named volume'. If you now do a ``docker volume ls``, you can see it has a volume name instead of a massive random string. 
- If you now remove the container ``docker container rm -f mysqlVolume``, and ``docker volume ls`` you can see its still there in the volume. Now if you run ``docker container run -d --name mysqlVolume2 -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql-db:/var/lib/mysql mysql``, no new volume will be created! It will instead attach itself to the existing volume with that name. You can use ``docker container inspect mysqlVolume2`` to see this in "Mounts" => note that the source is friendlier too
- Best to name volumes for the projects so that you know what they're for and that they need to stick around!
- You can create a volume using the docker ``container run`` command at runtime like we did above, and also in the Dockerfile. But also you can create them using ``docker volume create``, but you would only do this if: You wanted to specify a different driver, or to specify driver options, or put labels on the volume.
- NOTE: Volumes live inside dockers VM (linux)

***Shell differences for Path Expension***
how to share files and directories between a host and a Docker container. 
One of the parts of the command line you'll need to type is the host file path you want to share.
- For PowerShell use: ${pwd} 
- For cmd.exe "Command Prompt use: %cd%
- Linux/macOS bash, sh, zsh, and Windows Docker Toolbox Quickstart Terminal use: $(pwd) 

Note, if you have spaces in your path, you'll usually need to quote the whole path in the docker command.

***Persistent Data: Bind Mounting***
- A bind mount is a map that maps the host file or directory to a container file or directory
- Basically just two locations pointing to the same file(s)
- Wont delete host files when you delete container. If theres a conflict between host file and container file, the host files will win and overwrite any in the container. 
- Can't use in Dockerfile, must be a container run
- ... run -v /Users/Mark/stuff:/path/container (mac / linux) or ... run -v //c//Users/Mark/stuff:/path/container (windows)
- ``docker container run -d --name nginx4 -p 80:80 -v $(pwd):/usr/share/nginx/html nginx`` => for the $(pwd) command to work, you need to be in the right file, aka the directory you want to copy accross. i.e. udemy-docker-mastery/dockerfile-sample-2. If you then add a file to this directory, and get into the BASH of the container you just made (ie.) ``docker container exec -it nginx4 bash`` and cd into usr/share/nginx/html and give it an ls -a, you can see your updates on your local reflect whats in the container!

---

Assignment:
- Create a postgres container with named volume "psql-data" using version 9.6.1 (old)
- use dockerhub to learn volume path and versions needed to run it 
- check logs, stop container (when startup logs finished)
- create new postgres container with same named volume using 9.6.2
- check logs to validate (should be less since its not making the named volume)

Creating first container:
``docker container run -d --name postgres1 -e POSTGRES_PASSWORD=password -v psql-data:/var/lib/postgresql/data postgres:9.6.1``
Check logs:
``docker container logs postgres1``
Stop container: 
``docker container stop postgres1``
Start 2nd container: 
``docker container run -d --name postgres2 -e POSTGRES_PASSWORD=password -v psql-data:/var/lib/postgresql/data postgres:9.6.2``
Check logs: 
``docker container logs postgres2``

---

Assignment 2: Bind Mounts
- use a jekkyl 'static site generator' to start a local web server
- changing code on the host and watching it be reflected in the container (i.e. html etc)
- source code in course repo under bindmount-sample-1 (edit these on host)
- container detects changes and updates web server
- start container with: 
``docker run -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve`` (or 81:4000 if 80 is taken)
- change file in _posts directory, see the changes in the browser!s

===

postgres now needs a password passed in like so: (env variable) POSTGRES_PASSWORD=mypasswd

=== 

## Section 6: Making it easier with docker Compose: the multi-container tool

Files used in docker-compose will be on the same docker network!

**docker-compose**
- Why? configure relationships between containers
- Why: Save our docker container run settings in easy to read file
- Why: create a one-liner developer environment startups
Comprised of 2 seperate but related things
1. YAML-formatted file that describes our solution options for:
  - containers
  - networks
  - volumes
2. A CLI tool docker-compose used for local dev/test automation with those YAML files

**docker-compose.yml**
- compose YAML format has its own versions, recommend at least 2. Need to specify it or it defaults to 1! (you will lose features)
- YAML file can be used with docker-compose command for local docker automaion or..
- With docker directly in production with Swam (as of v.1.13)
- ``docker-compose --help``

heirachical: 
in YAML things need to be formatted a certain way, usually minimum 2 spaces etc.
"services" (which usually means each container)
- everything in docker run command goes in services basically - the first one under "services" is the container name!
- this docker-compose.yaml file basically is a shell script for running docker stuff
"volumes" (OPTIONAL)
- same as "docker volume create" or "docker network create" (OPTIONAL)

===

**Trying out basic compose commands w/ docker-compose cli**
- comes bundled with docker for windows/mac
- not a production-grade tool, ideal for local development and test
- two most common commands are:
  - ``docker compose up`` (setup volumes/networks and start all containers)
  - ``docker compose down`` (stop all containers and remove cont/vol/net)
- if all your projects had a Dockerfile and docker-compose.yml then the "new developer onboarding" would be:
    - ``git clone github.com/some/software``
    - ``docker compose up``

try out compose-assignment-2: 
cd into it and run ``docker-compose up``
control+c to stop server
to run it in the background: ``docker-compuse up -d``

a lot of the commands you're used to in docker are also in docker-compose.
docker-compose is just talking to docker cli in the background!

=== 

Assignment: Writing a compose file 
- build a basic compose file for drupal content management system website. 
- lookup docker hub for drupal and postgres images
- use ports to expose drupal on 8080
- use version 2 
- 2 services: one for drupal, one for postgres
- set POSTGRES_PASSWORD for postgres
- wlak through drupal setup via browser
- tip: drupal assumes DB is localhost, but its service name 
- extra credit: use volumes to store drupal inique data
https://hub.docker.com/_/drupal

(saved in assigments/docker-compose.yml)

===

Using Compose to build (Adding image building to compose files)
- Compose can also build your custom images
- will look in the cache to build them, or will build with docker-compose up if it does't find it 
- will need to rebuild with "docker-compose build" if it doesn't find it!
- great for complex builds that have lots of vars or build args

i.e. instead of this: 
````yml
services:
  proxy:
    image: nginx:1.13 # this will use the latest version of 1.13.x
    ports:
      - '80:80' # expose 80 on host and sent to 80 in container
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
````

it looks like this:
````yml
services:
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    ports:
      - '80:80'
````

instead of being given an image, its being given a build with a path (context) and a dockerfile!

=> go into compose-sample-3 and run "docker-compose up", can now edit the html file and refresh to changes (its bind mounted)
=> to stop this its not just ``docker-compose down`` (that will work but wont remove the image). To remove the image also you need to type ``docker-compose down --rmi local``

===

Assignment: Build and run compose
- "Building custom drupal image for local testing"
- start with docker-compose from last assignment
- 

Dockerfile:
the "\" in a dockerfile means include the next line in this command.
the "&&" means, if the first command succeeds, then run this one

docker-compose:
if at any point there is an image key and a build key, it changes the purpose of the image key. with a build, the image key is now saying its name. You can leave it out if you use just "build"!

===

## Section 7: Swam intro and creating a 3-Node Swarm Cluster

Containers everywhere = new problems
How do we deploy / maintain all these containers?
- How do we automate container lifecycle?
- How can we easily scale out/in/up/down?
- How can we ensure containers are re-created if they fail?
- How can we replace containers without downtime (i.e. if theres an update) => blue/green deploy?
- How can we control/track where containers get started?
- How can we create cross-node virtual networks?
- How can we ensure only trusted servers run our containers?
- How can we store secrets, keys, passwords, and get them to the right container (and ony that container?)

Swarm Mode: Built-in Orchestration
- A server clustering solution built inside docker
- Added in summer 2016, enhanced in 2017
- Not enabled by default, new commands once enabled 
  - ``docker swarm``
  - ``docker node``
  - ``docker service``
  - ``docker stack``
  - ``docker secret``
- split between "worker" and "managers". A manager is like a worker with permissions to control the swarm
- docker run just deploys one container
- a swarm manager might have 3 nginx replicas, and three workers (each one is a nginx container)
- managers issue orders down to the workers. managers themselves can also be workers

- a manager is a worker with permissions to control the swarm
- you can promote and demote managers to workers and workers to managers

- ``docker service`` command replaces ``docker run`` command (for a swarm). allows us to add extra features to a container when we run it - i.e. replicas are known as tasks 

===

**Create your first service and scale it locally**

- you can create a single node swarm
- to know if swarm is on or not, ``docker info``, it will say "swarm inactive" if not on (its off by default!)
- ``docker swarm init`` will give a single node swarm - swarm is now active
- docker swarm init out of the box =>
  - lots of PKI and security automation
    - root signing certificate create for our swarm
    - certificate issued for first manager node
    - join tokens are create
  - raft database created to store root CA, configs and secrets
    - encrypted by default on disk (1.13+)
    - no need for another key/value system to hold orchestration/secrets
    - replicates logs amongst managers via mutual TLS in "control plane"

too add a worker to the swarm: 
``docker node ls`` => see the nodes you've created
``docker node --help``
``docker swarm COMMAND`` 

``docker service --help`` => replaces docker run
``docker service create alpine ping 8.8.8.8`` => spits back a random id (the service id)
``docker service ls`` to see it. this doesn't show us the actual container, just a list of the services. 
to see actual container: ``docker service ps <name or id>``. it now has a node component
``docker container ls`` still works, but docker is adding some info to the name etc
to scale up service: ``docker service update <name or id> --replicas 3``

if you try to remove a container (i.e. one of the replicas), with something like ``docker container rm -f <name>``,then you check ``docker service ls`` you'll now have 2/3. if you check ``docker service ls`` again after a few seconds, you'll see its back to 3/3. The goal of an orchastration system like swarm is to keep up the servers and replace them if they go down (much unline a standard docker container run command).
If you want to take these containers down, you need to take the orchastation service down: ``docker service rm <id or name of service>``

===

Creatign a 3-node swarm cluster:
play-with-docker.com (because we need 3 OS's)

``docker swarm init`` doesnt worm (common on cloud servers) => it wants us to specify an ip address to advertise the swarm ip on. You need one accessible from other servers:
``docker swarm init --advertise-addr <ip address>``

now need to copy the command it spits out: 
``docker swarm join --token SWMTKN-1-16bzmi3st89fuem4r6384jmwr73kbx5tadiz3gqdqb1tb0v3cx-5iasq3zkjxfatg0ner5f2e613 192.168.0.18:2377``
paste this into node2

now go back to node1, ``docker node ls`` and you can see node2 has joined. note 2nd node is only a worker. 
on node2 you can't use "docker node ls", because its only a worker. If you want to update node2 to be a manager, you need to use node1 ``docker node update --role manager node2``. Now in ``docker node ls`` its manager status is "reachable".

now to add node3, type ``docker swarm join-token <manager or worker>``

now in node 1: ``docker service create --replicas 3 alpine ping 8.8.8.8``
now can use ``docker service ls`` => 3 or 3
 ``docker node ps`` => see local node running the container
 , ``docker service ps <name or id>``

If setup like this, you can really operate the whole thing from node1

===

Section 8: Swarm Basic Features and how to use them in your workflow 

Gonna park this here. Haven't had to do swarm / k8s stuff yet... no use case