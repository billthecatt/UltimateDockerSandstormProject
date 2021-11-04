# Why? 

There are three main docker repos that have the most reviews and usage. Each of them appears to have their own unique reason to exist and I'd like to encorprate the functionality from all of them into one, "best of all worlds" docker instance. Also, I'd like to develop an instance build script that allows us to easily create a cloud instance so that the docker repo we create can move easily between AWS, Azure and GCP. This will also help facilitate the testing of the different instance offerings of each provider and allow us to easily refactor the instance vertically, since horizontal scaling doesn't work well for game server instances. 

## Existing work:
The docs from amazon on getting docker working on an ec2 instance are here:

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html

The three docker repos that we want to replicate functionality from are as follows:

1. https://hub.docker.com/r/andrewmhub/insurgency-sandstorm
2. https://hub.docker.com/r/snickch/insurgencysandstormdedicatedserver
3. https://hub.docker.com/r/taxx/sandstorm-lgsm

### The benefits of each are as follows:

1. This docker image appears to contain support for the ISMC mod but appears to not completely do things the "docker way" in that, it isn't expected that you'll be oftend downloading a new docker image and refactoring. As such the config files are stored on mounts and updating steam appears to be a thing you get to do on your own. Additionally, this image uses a rather large (2.98G) Ubuntu package as it's baseline instead of my perfered, smaller, debian image, which the other two docker projects use. As such, we may end up using the ISMC specific stuff from this project and leaving the rest. The first time I built this on a t1micro amazon image I overloaded the default 8g storage, so we'll need to experiement to see what it really needs for storage. 
3. This project does things the "docker way" in that, it's rebuilt daily and designed to be burned easily and rebuilt, and also uses a smaller debian image. This is missing the ISMC mod and the LGSM support we know and love for server administration via tmux. This may be the repo we use the most from, but we'll need to integrate ISMC and LGSM from the other two projects if that's not too hard to add on easily.
4. This project makes LGSM work out of the box, but doesn't do as good a job on the "docker way" as project #2 does. We may just use the LGSM functionality code from this and move on. 

One of the design goals of all of this is to make the AMI (or resultant docker) so easily deployable that we can migrate it between AWS, Asure and/or GCP. Additionally, I'd like to see if we can make it use cloud hosted (GitHub?) configuration files so that we can use AWS spot instances and save 90% on the instance cost since these will be on demand instances that we only use occasionally during a LAN. 


#### Getting the AndrewMarchukov docker working:

In spite of this being a docker and supposedly standalone, the docker needs some config files ready to go on the "host OS" that we'll need to get from the cloud before it's ready to launch. 

As such, here is the script to download all of that from the cloud and put it where the launch script expects it to be (should we integrate this into the AMI build script so it's all ready to go?):

```
#/bin/bash
sudo mkdir -p /home/user/coop-modmap
sudo mkdir -p /home/user/coop-modmap/Mods
sudo mkdir -p /home/user/coop-modmap/config/ini
sudo mkdir -p /home/user/coop-modmap/config/txt

#this is the stock modmap.env from his project
#sudo wget --no-check-certificate -O /home/user/coop-modmap/modmap.env https://github.com/AndrewMarchukov/insurgency-sandstorm-server-dockerize/raw/master/modmap.env

#This is our custom one...
sudo wget --no-check-certificate -O /home/user/coop-modmap/modmap.env https://raw.githubusercontent.com/billthecatt/UltimateDockerSandstormProject/main/modmap.env 
sudo wget --no-check-certificate -O /home/user/coop-modmap/config/ini/Engine.ini https://github.com/AndrewMarchukov/insurgency-sandstorm-server-dockerize/raw/master/config/ini/Engine.ini
sudo wget --no-check-certificate -O /home/user/coop-modmap/config/txt/Game.ini https://github.com/AndrewMarchukov/insurgency-sandstorm-server-dockerize/raw/master/config/ini/Game.ini
# prereqs setup now we can pull the docker image and not have it barf:
docker pull andrewmhub/insurgency-sandstorm
```

The launch script for this docker is as follows, taken verbatim from the project:

```
docker run -d --restart always --env-file /home/user/coop-modmap/modmap.env \
--name sandstorm-modmap --net=host \
-v /home/user/coop-modmap/Mods:/home/steam/steamcmd/sandstorm/Insurgency/Mods:rw \
-v /home/user/coop-modmap/config/ini:/home/steam/steamcmd/sandstorm/Insurgency/Saved/Config/LinuxServer:ro \
-v /home/user/coop-modmap/config/txt:/home/steam/steamcmd/sandstorm/Insurgency/Config/Server:ro andrewmhub/insurgency-sandstorm:latest
```

Evidently this is too much for a t1micro instance because shortly after docker launch, I'm seeing the following errors on even the most basic docker command:
```
[ec2-user@ip-172-31-80-197 ~]$ docker stop
fatal error: runtime: out of memory
```
As a result, we'll need to research ways to slim down the memory requirements and/or research a better EC2 instance type. ( t1micro instances have 1G of ram by default and if we change this, we'll incur additonal costs. ) We'll continue testing w/ the other docker projects to see if they are lighter weight...

Testing with a t2.medium (2vcpu, 4g ram, 30g storage) appears to support docker better so far.. We'll see how well it supports the full install of SandStorm..

Once you get the docker instance launched correctly you can connect to it via bash as follows:
```
docker container exec -it sandstorm-modmap /bin/bash
```
If you wish to stop the docker instance use the following:
```
docker stop sandstorm-modmap
```
If you want to start it after having stopped it, use the following:
```
docker start sandstorm-modmap
```
If you want to change anything in the modmap.env file, this gets built into the container at instantiation and can't be changed on the fly, even if you modify the local file. So you have to remove the docker container and start all over, as follows:
```
docker stop sandstorm-modmap
docker rm sandstorm-modmap
docker run <SYNTAX> #use the syntax above to run the new container
```
Interestingly enough, this instance provides no way to connect to the running console or validate that it's launched correctly. (Other than ps -ef and checking for the running server instance.) This leaves much to be desired, but appears to at least launch and not crash under a t2.medium instance. I'll need to do some playtesting to ensure it doesn't suck..

#### Getting the SnickCH docker working:

```
docker pull snickch/insurgencysandstormdedicatedserver
```

While this server seems to be lighter weight than the other, 1G is certianly not enough ram for these instances, with docker running as an abstraction layer between the OS and the proces..


