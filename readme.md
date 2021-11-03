This is the start of this project. 

There are three main docker repos that have the most reviews and usage. Sadly each of them has their own unique reason to exist and I'd like to encorprate the functionality
from all of them into one, "best of all worlds" docker instance. Also, I'd like to develop an instance build script that allows us to easily create a cloud instance so that
the docker repo we create can move easily between AWS, Azure and GCP. This will also help facilitate the testing of the different instance offerings of each provider and allow
us to easily refactor the instance vertically, since horizontal scaling doesn't work well for game server instances. 

The docs from amazon on getting docker working on an ec2 instance are here:

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html

The three docker repos that we want to replicate functionality from are as follows:

1. https://hub.docker.com/r/andrewmhub/insurgency-sandstorm
2. https://hub.docker.com/r/snickch/insurgencysandstormdedicatedserver
3. https://hub.docker.com/r/taxx/sandstorm-lgsm

The benefits of each are as follows:

1. This docker image appears to contain support for the ISMC mod but appears to not completely do things the "docker way" in that, it isn't expected that you'll be oftend downloading a new docker image and refactoring. As such the config files are stored on mounts and updating steam appears to be a thing you get to do on your own. Additionally, this image uses a rather large (2.98G) Ubuntu package as it's baseline instead of my perfered, smaller, debian image, which the other two docker projects use. As such, we may end up using the ISMC specific stuff from this project and leaving the rest. The first time I built this on a t1micro amazon image I overloaded the default 8g storage, so we'll need to experiement to see what it really needs for storage. 
3. This project does things the "docker way" in that, it's rebuilt daily and designed to be burned easily and rebuilt, and also uses a smaller debian image. This is missing the ISMC mod and the LGSM support we know and love for server administration via tmux. This may be the repo we use the most from, but we'll need to integrate ISMC and LGSM from the other two projects if that's not too hard to add on easily.
4. This project makes LGSM work out of the box, but doesn't do as good a job on the "docker way" as project #2 does. We may just use the LGSM functionality code from this and move on. 

One of the design goals of all of this is to make the AMI (or resultant docker) so easily deployable that we can migrate it between AWS, Asure and/or GCP. Additionally, I'd like to see if we can make it use cloud hosted (GitHub?) configuration files so that we can use AWS spot instances and save 90% on the instance cost since these will be on demand instances that we only use occasionally during a LAN. 



