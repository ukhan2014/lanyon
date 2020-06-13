---
layout: note
title: phpfpm service on docker 
---

While trying to install wordpress on a docker container it was noticed that when the docker container is started, none of the services come up automatically. Specifically nginx and phpfpm were not up. This caused wordpress not to get served.

First check if php has any running process:
{% highlight js %}
root@bd8c20437f12:/etc/nginx/sites-available# ps -aux | grep php
root        89  0.0  0.0   3304   664 pts/2    S+   13:26   0:00 grep --color=auto php
{% endhighlight %}

This is just grep, not php. So we try to see if we can find the php service:      
{% highlight js %}
root@bd8c20437f12:/etc/nginx/sites-available# service php-fpm7.4 status
php-fpm7.4: unrecognized service
{% endhighlight %}

Seems like we don't know the name of service. So we try to find it:
{% highlight js %}
root@bd8c20437f12:/etc/nginx/sites-available# service --status-all
 [ - ]  dbus
 [ ? ]  hwclock.sh
 [ - ]  mysql
 [ + ]  nginx
 [ - ]  php7.4-fpm
 [ - ]  procps
 [ - ]  rsync
{% endhighlight %}

Above we can see that the process name we tried was incorrect. The actual name is php7.4-fpm, and the minus sign indicates that it is not running. We can confirm this by doing:
{% highlight js %}
root@bd8c20437f12:/etc/nginx/sites-available# service php7.4-fpm status 
 * php-fpm7.4 is not running
{% endhighlight %}

Since we want to start it, we can do:
{% highlight js %}
root@bd8c20437f12:/etc/nginx/sites-available# service php7.4-fpm start
{% endhighlight %}

Let's check if it started:
{% highlight js %}
root@bd8c20437f12:/# service php7.4-fpm status
 * php-fpm7.4 is running
{% endhighlight %}

Now we are good to go. Now we can continue with our work on wordpress.

