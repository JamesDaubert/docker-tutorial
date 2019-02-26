# Docker and AWS Tutorial
## CS308

## An Introduction to Docker
Docker is:
> an open-source project that automates the deployment of software 
> applications inside containers by providing an additional layer 
> of abstraction and automation of OS-level virtualization on Linux.

This is a fancy way of saying that Docker provides a sandbox type of environment for deployment of applications, in the form of a Docker _container_. A container can be standardized and shipped out to many different machines are once, requiring little or no set-up to organize once the Docker framework is laid out. Docker is used by a variety of companies to:
* Increase developer productivity
* Release software at a faster rate
* Reduce the need for IT infrastructure
* Speed up deployment

## Getting Started with Docker
Let's first install Docker. Download an installer for [Mac](https://www.docker.com/products/docker-engine#/mac), [Linux](https://www.docker.com/products/docker-engine#/linux), or [Windows](https://www.docker.com/products/docker-engine#/windows). Once the installation is complete, make sure that Docker is installed on your command line by running the following command:

`docker run hello-world`

`> Hello from Docker....`

Congrats! You officially have Docker installed.

## Busybox
We will now learn more about Docker by running a [Busybox](https://en.wikipedia.org/wiki/BusyBox) container. Busybox provides several Unix utilities in a single source, giving us plenty of built-in functionality with which to work. To get started, fetch the busybox image from the Docker registry:

`docker pull busybox`

By running this command, we retrieve a local version of Busybox to launch as a container on our system. We can now test this container by typing:

`docker run busybox echo "I love Busybox!"`

By passing a command, we instruct the container to  launch, execute our command, from within the Busybox container and then exit. This all happens in a split second!

_So why run our command through a docker container? Isn't this just extra work?_ The answer to this question becomes more apparent as we advance to more complex applications. 

## Docker and Webapps 
We are now going to use Docker to deploy a static website. Navigate to the `static-webapp` directory in this tutorial repository and explore the files there. 

The `html` folder holds all of the static content to be served on the website (including HTML). The `Dockerfile` is of particular importance -- this file defines the base image for our Docker container. Because we are running a very simple application, all Dockerfile is mainly composed of the Alpine version of Nginx, which lets us deploy static HTML.

To build our static HTML image, run the following command:

`docker build -t webserver-image:v1 .`

This command builds and configures a docker container. We can now launch this container on host port and container port 80 with the command:

`docker run -d -p 80:80 webserver-image:v1`

If you visit the page localhost:80, you should now see the static webpage! 

### Making Changes
Try following the instructions listed on the static webpage. Once you have made these changes in the `static-webapp` folder, run `docker ps -a` and find the docker container with the status 'Up.'  Run the following commands to get a fresh start:

`docker stop <container_id>`

`docker rm <container_id>`

There are more efficient ways to do this, but we will stick with this for ease of understanding. Now you can rebuild and launch an entirely fresh container with your new changes. To do all of these tasks quickly, run the `docker_reset` script.

### Why does this help us?
To see the difference between your local computer and the container, run the following commands:

`docker run webserver-image:v1 nginx -v`

`nginx -v`

Unless you already had nginx installed on your computer, you should observe that the first command prints out a nginx version while the latter does not. This is because all of the configuration was accomplished on the docker container, rather than your local computer. When you kill this container, the configuration will go with it. With more advanced webpages, we may need a variety of resources to be installed on the container. Docker handles all of this configuration without modifying your local computer settings.

## Useful Docker commands:

* `docker ps`: List all running containers.
* `docker ps -a`: List all containers.
* `docker rm <container ID>`: Delete docker container.
* `docker rm $(docker ps -a -q -f status=exited)`: Delete all exited containers.

## Terminology:
* Images - The blueprints of our application which form the basis of containers. In the demo above, we used the `docker pull` command to download the busybox image.
* Containers - Created from Docker images and run the actual application. We create a container using docker run which we did using the busybox image that we downloaded. A list of running containers can be seen using the `docker ps` command.
* Docker Daemon - The background service running on the host that manages building, running and distributing Docker containers. The daemon is the process that runs in the operating system to which clients talk to.
* Docker Client - The command line tool that allows the user to interact with the daemon. 
* Docker Hub - A registry of Docker images. You can think of the registry as a directory of all available Docker images. 

## AWS 
### Getting started with EC2
EC2, which stands for Elastic Compute Cloud, is a service which provides virtual machines for users to run their own applications on the cloud. It forms the bedrock for many other AWS services, such as AWS Elastic Beanstalk, which will be used later in the tutorial. 
#### Creating an instance
Navigate to the [AWS Console](console.aws.amazon.com) and click on EC2 from the "Compute" menu. Then click Launch Instance to begin. First you will be prompted to choose an OS, for this tutorial keep the default selection of Amazon Linux 2 AMI, click Select. Then you will be prompted to choose an instance type, once again keep the deafult of t2.micro, which will provide enough storage and performance. Then click "Launch". You will then be prompted to create a new key pair to authenticate yourself to the servers. This will allow you to access the servers from your local machine's console, while not allowing anyone without the private key access. Download your key pair, and wait for your server to launch. 
#### Connecting to your EC2 instance
Now that the server is up and running, you will connect to it from your local machine. Ensure you have SSH by typing ssh into the command line of your preferred console (the default being Terminal for Mac users). Change directories to the folder containing the .pem file with the key pair (do this by typing `<cd Documents/Tutorial>` for example). You will need to change permissions of the private key so you can read it, do this by typing `<chmod 400 _______.pem>` Then you are ready to connect to your instance! Then on your command line you must type ssh -i /path/my-key-pair.pem user_name@public_dns_name. Plug in the proper values. For user_name it is probably ec2-user, and you can find the public_dns_name in the EC2 console. The console may respond asking if you wish to continue connecting because authenticity can't be established, type yes. Now you are connected to your EC2 instance, and can use it just as you use your local machine.
### Elastic Beanstalk
Now that you are familiar with the standard EC2 instance that AWS offers, let's get more robust. Elastic Beanstalk (EBS) is an AWS PaaS that allows you to upload and deploy applications, and run them on top of an EC2 instance. EBS allows for many different types of applications to be run on it, but as we have already created a containerized web app from Docker in the last section, why not continue with that example and use Docker as our configuration. Per AWS’s docs: 
>By using Docker with Elastic Beanstalk, you have an infrastructure that automatically handles the details of capacity provisioning, load >balancing, scaling, and application health monitoring.
This is great! By using EBS with Docker we are alleviating much of the stress that is brought with utilizing servers to do our processing. 
#### Updating the Webapp
Before you can upload your webapp to the AWS console, you need to add a key line to your Dockerfile. Open it in your preferred text editor and add the line `EXPOSE 80` after the other instructions. This tells the Dockerfile to expose the port on 80 to the AWS server so that it can be accessed. WIthout this line you would get a 502 Bad Gateway Error, because the server would not be able to actually access its content. Now that your webapp is ready for EBS deployment, you just need to zip the files. It is important to not zip the parent folder 'static-webapp', but to navigate into it and highlight the three files, control click, and compress them. This will create the zip that is needed to comply with the AWS system. If you compress the static-webapp folder the system will not be able to access the Dockerfile for instructions on deployment.
#### Deploying to EBS 
Once you have your Dockerfile files ready for use, uploading the webapp to the cloud is very simple. Open the AWS EBS console, and click Create New Application. Put in an application name like "AWS-Tutorial" and click Create. Then create a new environment in this application. Choose web server environment, and choose Docker from the preconfigured platform. In the Application code section, choose "upload your code", and select the .zip file containing your web app. Then click create. It will take a few minutes to launch, but once complete, you should be able to return to the environment page, and click on the url that now has the live version of your webapp! 

## Sources:

* https://docker-curriculum.com/
* https://www.katacoda.com/courses/docker/create-nginx-static-web-server
* https://github.com/prakhar1989/docker-curriculum/blob/master/static-site/html/index.html
* https://stackoverflow.com/questions/47428504/javascript-count-number-of-visitor-for-website
* https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/single-container-docker.html
