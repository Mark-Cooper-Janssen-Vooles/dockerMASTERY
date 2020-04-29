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