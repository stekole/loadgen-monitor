# loadgen-monitor
A vagrant instance running a load generator and a container monitor (c-advisor)

Author: Stefan Kolesnikowicz 


## Setup:

    Host: Mac osx 
    Hypervisor: Virtualbox Version 5.0.24 r108355
    Vagrant Version 1.8.5
    Vagrant “Hypervisor” Host: Centos7
    Cloud-init Version: cloud-init.x86_64 0:0.7.5-10.el7.centos.1
    Docker Version : docker.x86 1.12.6-32.git88a4867.el7.centos                               


## Introduction


This project is broken down into three parts just to walk you, or anyone, through 
the build process:

The three parts are broken down into:

Part 1: Vagrant

Part 2: Containers

Part 3: Monitoring 


## Part 1 - Vagrant
This section you will be setting up your "hypervisor" which will be running your containers. 

First close the vagrant repo to pull a bare vagrant box with cloud-init running

    git clone https://github.com/stekole/vagrant-box

    cd vagrant-box && vagrant up


Wait until the server is provisioned 


You should see the following if everything was successful:

    hypervisor.local: + touch /opt/.vagrant_provision_success
    hypervisor.local: + service docker start
    hypervisor.local: Redirecting to /bin/systemctl start  docker.service
    hypervisor.local: + exit 0


## Part 2 - Containers
Building a container running a simple load generator script. 

Login to your vagrant instance now using the vagrant ssh command:
    
    vagrant ssh 
   
Pull down the Dockerfile for our load generator:
    
    git clone https://github.com/stekole/docker-loadgenerator &&   cd docker-loadgenerator 

And lets build the container from the docker file

    sudo docker build -t loadtest:latest .
    Successfully built

Once your docker image has been created, you can run it by its image id

Use the docker images command to find the ID, here is a quick one-liner

    ID=`sudo docker images | grep load | awk '{print $3}'` && sudo docker run -i -d $ID

Check if it is running with
     
    sudo docker ps 


## Part 3 - Monitoring (Container B)


Utilizing an opensource solution, I decided to choose cadvisor as it is small/reliable and can be quickly setup. A simple container as we are not doing anything very complicated in scale (and just in vagrant). 

Cadvisor is a application by google to analyzes and graph resource usage and performance characteristics of running containers. This allows us to look at not only the hypervisor but our containers

Lets download it and start it

    sudo docker run   --volume=/:/rootfs:ro   --volume=/var/run:/var/run:rw   --volume=/sys:/sys:ro   --volume=/var/lib/docker/:/var/lib/docker:ro   --publish=8080:8080   --detach=true  --privileged=true --name=cadvisor   google/cadvisor:latest

You are able to use your desktop machine hypervisor on port 4444
    
    http://localhost:4444/containers/

Individual containers can be viewed by clicking the top link on the url above or visiting the following link:

    http://localhost:4444/docker/


