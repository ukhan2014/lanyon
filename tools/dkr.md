---
layout: note
title:  Creating/using docker containers
---

A container is a sandboxed rootfs that can be modified and saved as an image. You can set your application and separate your machine's resources from other containerized applications you may need to run. 

To create an Ubuntu image from scratch:
{% highlight js %}
sudo docker run -it ubuntu bash
{% endhighlight %}

To view docker images you have created:
{% highlight js %}
sudo docker images
{% endhighlight %}

To start a new container with name "ContainerName":
{% highlight js %}
sudo docker exec -it containerName
{% endhighlight %}

To start a container with name "ContainerName" and bind port 8080 of container to  port 80 of host system:
{% highlight js %}
sudo docker run --name containerName -p 8080:80 ubuntu
{% endhighlight %}

Then to run a command inside the container (from the host machine shell):
{% highlight js %}
docker exec -it <container_id_or_name> echo "hello!"
{% endhighlight %}

To see all docker containers that you have previously created (started and stopped):
{% highlight js %}
sudo docker ps -a
{% endhighlight %}

This should give the following type of output:
{% highlight js %}
CONTAINER ID   IMAGE    COMMAND    CREATED       STATUS              PORTS                  NAMES
094fcef21010   ubuntu    "bash"    2 days ago    Up About a minute   0.0.0.0:8080->80/tcp   wpress
{% endhighlight %}

If you stop a container, services will not be up next time you start the container:
{% highlight js %}
root@094fcef21010:/# service nginx status
 * nginx is not running
{% endhighlight %}

To make sure the service you need starts up, give this command after starting the container:
{% highlight js %}
$ sudo docker exec -it wpress service nginx start
 * Starting nginx nginx                                                                     [ OK ]
$ sudo docker exec -it wpress service php7.4-fpm start
$ sudo docker exec -it wpress service php7.4-fpm status
 * php-fpm7.4 is running
{% endhighlight %}

To create an image out of a running container:
First get the ID of the container using:
{% highlight js %}
sudo docker ps
{% endhighlight %}

Then commit the changes into an image:
{% highlight js %}
sudo docker commit <containerId> <nameOfNewImage>
{% endhighlight %}

Now if you check the images, your new image will be part of the list:
{% highlight js %}
$:~/Docker$ sudo docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
containerId    latest    714f2314e9ec   11 minutes ago   690MB
<none>         <none>    796136e6d1a7   3 days ago       227MB
ubuntu         latest    1d622ef86b13   2 weeks ago      73.9MB
hello-world    latest    bf756fb1ae65   4 months ago     13.3kB
{% endhighlight %}

Once you have some docker images you can launch containers from them using the following command:

{% highlight js %}
sudo docker run -itd --name <container_name> -p 8084:80 -p 8085:443 <docker_image_name> bash
{% endhighlight %}

The -itd flags are important:
  -it means start the docker container as INTERACTIVE
  -d is for starting the docker container in DETACHED mode

The -p flag is also important. Make sure you expose the ports required for your application. In the above example port 8084 of the host machine is attached to port 80 on the docker container and port 8085 is attached to port 443 of the docker container:

{% highlight js %}
   Host(physical server)     Target(docker container)
            8084                      80
            8085                      443
{% endhighlight %}
