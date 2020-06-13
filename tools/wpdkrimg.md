---
layout: note
title:  Using saved docker image of wordpress (wp-final)
---

The image is committed on the bearcloud box.

First create a wordpress container from the saved image wpress-final. Make sure you modify CONTAINER_NAME and the ports after the -p hook. 8080 is the port on the host server, and 80 is the port that is mapped to it inside the container.
{% highlight js %}
sudo docker run -itd --name wpress_CONTAINER_NAME -p 8080:80 -p 8081:443 wpress-final bash
{% endhighlight %}

Using the host server ports that you have linked to the container above, go to the main gateway reverse proxy server and set up the server configs in the nginx configuration. Use one of the previously set up wordpress configs as an example. You will also need to make a copy of the ssl.conf file for the new domain. We are talking about 2 steps here:
1. Copy the reverse proxy virtual host config for the new domain
2. Copy the corresponding ssl.conf file for the new config

After this, you will have to get certificates for those server hosts. For example if you want the domain container.domain.com you have to go to the config directory in the gateway reverse proxy and use the dehydrated script:

First make sure you are pointing to the staging server for LetsEncrypt:
{% highlight js %}
vi config <-- edit the config file for the dehydrated script
{% endhighlight %}

Make sure the following line is uncommented by taking the hash away:
{% highlight js %}
#CA="https://acme-staging-v02.api.letsencrypt.org/directory"
{% endhighlight %}

Then do:
{% highlight js %}
bash ./dehydrated --domain container.domain.com --cron
{% endhighlight %}

This should get the certificates and put them under ./certs/container.domain.com directory. If this results in success, then you can comment out the staging server line in the config file, remove the staging certificates directory ./certs/container.domain.com and run the above command again to get real certificates.

Now log on to the server with the containers and get into docker container by doing:
{% highlight js %}
sudo docker exec -it <wordpress_container> bash
{% endhighlight %}

First check if you can access the wordpress site from localhost in the container itself. You will not be able to:
{% highlight js %}
root# curl localhost:443
curl: (7) Failed to connect to localhost port 443: Connection refused
{% endhighlight %}

Then check the status of services that are required. The required services are mysql, nginx and php. They will all be offline to start:
{% highlight js %}
root# service --status-all
 [ - ]  dbus
 [ ? ]  hwclock.sh
 [ - ]  mysql
 [ - ]  nginx
 [ - ]  php7.4-fpm
 [ - ]  procps
 [ - ]  rsync
 {% endhighlight %}

Start up the required services:
{% highlight js %}
root# service mysql start; service nginx start; service php7.4-fpm start
 * Starting MariaDB database server mysqld                                                                               [ OK ] 
 * Starting nginx nginx                                                                                                  [ OK ] 
{% endhighlight %}

Now check the status of the services again:
{% highlight js %}
root# service --status-all
 [ - ]  dbus
 [ ? ]  hwclock.sh
 [ + ]  mysql
 [ + ]  nginx
 [ + ]  php7.4-fpm
 [ - ]  procps
 [ - ]  rsync
{% endhighlight %}

Now check if you can access the wordpress site from inside the container. Execute:
{% highlight js %}
curl localhost:443
... lots of html should get printed
<script type='text/javascript' src='https://container.domain.com/wp-content/plugins/elementor/assets/js/frontend.min.js?ver=2.9.9'></script>
</body>
</html>
{% endhighlight %}


To change username and password:

Log into mysql:
{% highlight js %}
mysql -u root
{% endhighlight %}

Switch to the wordpress database:
{% highlight js %}
use wpdb; 
{% endhighlight %}

Take a look at the users table
{% highlight js %}
SELECT ID, user_login, user_pass FROM wp_users;
{% endhighlight %}

Update the username for user with ID = 1 in the users table:
{% highlight js %}
UPDATE wpress_users SET user_login='username' WHERE ID = 1;
{% endhighlight %}

Update the password for user with ID = 1 in the users table:
{% highlight js %}
UPDATE wpress_users SET user_pass=MD5('password') WHERE ID = 1;
{% endhighlight %}

Now from the container, manually change the wordpress config to allow access from the new domain:
{% highlight js %}
cd /var/www/wordpress
vim wp-config.php
{% endhighlight %}

Add the following lines:
{% highlight js %}
define( 'WP_HOME', 'https://container.domain.com' );
define( 'WP_SITEURL', 'https://container.domain.com' );
{% endhighlight %}

Now you can try to visit the wordpress instance by trying to access it from an external network, such as LTE on your phone. If the website loads, check to URL to make sure that the wordpress instance has not forwarded you to a different container's URL as the container was copied from a pre-existing image. If everything with the URl seems okay, you can try going to https://container.domain.com/wp-admin to login.

Once logged in, you can use the plugin called "Better Search and Replace" to replace all the old URLs with the new URLs. When all the URLs are updated and the plugin says 0 remain to be updated, then you can take the added lines out of wp-config.php as it is poor practice to have those over there and may cause issues. If the plugin cannot change any of the URLs and some are still coming up as unchanged, you can manually change the URLs in the mysql database. A common table that doesn't get updated is wp_options:

{% highlight js %}
mysql -u root;
use wpdb;
SELECT * FROM wp_options WHERE option_name='siteurl';  <-- This is the row you will need to modify.
{% endhighlight %}

After this, the login should be good and can be accessed at:
{% highlight js %}
https://container.domain.com/wp-admin
{% endhighlight %}

