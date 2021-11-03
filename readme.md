This is the start of this project. 

There are three main docker repos that have the most reviews and usage. Sadly each of them has their own unique reason to exist and I'd like to encorprate the functionality
from all of them into one, "best of all worlds" docker instance. Also, I'd like to develop an instance build script that allows us to easily create a cloud instance so that
the docker repo we create can move easily between AWS, Azure and GCP. This will also help facilitate the testing of the different instance offerings of each provider and allow
us to easily refactor the instance vertically, since horizontal scaling doesn't work well for game server instances. 

The docs from amazon on getting docker working on an ec2 instance are here:

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html



