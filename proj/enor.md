---
layout: note
title:  Installing Entware on AC-86u
---


1. Install Merlin Firmware First
2. Administration > System > Persistent JFFS partition settings:
   Disable "Format JFFS Partition at next boot"
   Enable "JFFS Custom scripts and configs"
3. enable SSH access by selecting "LAN Only", and enable HTTPS access to the interface. The port will switch from 80 to 8443 automatically
4. ssh into the router
5. type amtm and press i on the menu
6. Insert a USB 3 usb disk into the USB 3 slot. Do dmesg to check if disk was detected. If not, reboot the router. This USB disk will remain in the router. It is required to install and run entware and the entware packages such as nginx on the router
7. press fd becauase you need to format the disk. You will need to follow the prompts. entware requires ext2, ext3 or ext4. Pick ext4 for USB3
8. Install entware by doing amtm, pressing i, then pressing ep to install entware on the USB disk that you have inserted
9. After entware is installed, install the necessary packages for nginx and LetsEncrypt certificates:
{% highlight js %}
	opkg install bash ca-certificates coreutils-mktemp curl diffutils grep nginx-extras openssl-util
	{% endhighlight %}
10. Download helper script to get/renew Let's Encrypt certificates:
{% highlight js %}
	cd /opt/etc/nginx
	curl -O https://raw.githubusercontent.com/lukas2511/dehydrated/master/dehydrated
	{% endhighlight %}
11. Edit two sections at /opt/etc/nginx/nginx.conf:

	in server section:
	{% highlight js %}
	server_name "your.subdomain.tld";
	{% endhighlight %}

	in location section:
	{% highlight js %}
	root /opt/share/nginx/html;
	{% endhighlight %}

12. Now start nginx and try to access the domain name. The default nginx page should show up:
{% highlight js %}
    /opt/etc/init.d/S80nginx start
    {% endhighlight %}

13. Prepare the folder, which will be used by Let's Encrypt service to make sure this web server belongs to you. The following info is from https://github.com/Entware/Entware-ng/wiki/Using-Let%27s-Encrypt

{% highlight js %}
	echo WELLKNOWN="/opt/share/nginx/html/.well-known/acme-challenge" > config
	mkdir -p /opt/share/nginx/html/.well-known/acme-challenge
	{% endhighlight %}

	and start script to get new SSL sertificate:
{% highlight js %}	
	bash ./dehydrated --domain my.domain.ru --cron
{% endhighlight %}

	You'll get following answer if all goes fine:
	
{% highlight js %}
	/opt/etc/nginx # bash ./dehydrated --domain my.domain.ru --cron
	# INFO: Using main config file /opt/etc/nginx/config.sh
	Processing my.domain.ru
	 + Signing domains...
	 + Generating private key...
	 + Generating signing request...
	 + Requesting challenge for my.domain.ru...
	 + Responding to challenge for my.domain.ru...
	 + Challenge is valid!
	 + Requesting certificate...
	 + Checking certificate...
	 + Done!
	 + Creating fullchain.pem...
	 + Done!
	 	 {% endhighlight %}

{% highlight js %}
	 location ^~ /.well-known {
		allow all;
	 }
	 {% endhighlight %}

{% highlight js %}
	 openssl dhparam -out dhparams.pem 2048    <-- This command can take a long time depending on cpu power
	 {% endhighlight %}


	and put following content to /opt/etc/nginx/ssl.conf:

	{% highlight js %}
	ssl_certificate /opt/etc/nginx/certs/my.domain.ru/fullchain.pem;
	ssl_certificate_key /opt/etc/nginx/certs/my.domain.ru/privkey.pem;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GC	M-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE	-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-S	HA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES25	6-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
	ssl_prefer_server_ciphers on;
	ssl_dhparam /opt/etc/nginx/dhparams.pem;
	
	ssl_session_cache shared:SSL:10m;
	ssl_session_timeout 5m;
	# ssl_stapling on;
	
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
	{% endhighlight %}


	Restart web server and make sure https://my.domain.ru is available from internet:
	{% highlight js %}
	/opt/etc/init.d/S80nginx restart
	{% endhighlight %}

	By default, the Let's Encrypt SSL certificate is valid for 3 months. You can renew it from cron automatically:
{% highlight js %}
	opkg install cron
	{% endhighlight %}


	Create cron job by placing following script to /opt/etc/cron.monthly/renew_certs.sh:
{% highlight js %}
	#!/bin/sh
	cd /opt/etc/nginx
	bash ./dehydrated --domain my.domain.ru --cron
	nginx -s reload
	{% endhighlight %}

	and make it executable:

{% highlight js %}
	chmod +x /opt/etc/cron.monthly/renew_certs.sh
	{% endhighlight %}

	Don't forget to start cron:
{% highlight js %}
	/opt/etc/init.d/S10cron start	
	{% endhighlight %}

14. If your router reboots, your nginx will not auto start. A startup script needs to be placed under /jffs/scripts
    The post-mount script is created by entware. It waits for the USB disk to finish mounting and then runs the commands. Below is an example post-mount script:
{% highlight js %}
    admin@RT-AC86U-52E0:/jffs/scripts# cat post-mount 
	#!/bin/sh

	. /jffs/addons/diversion/mount-entware.div # Added by amtm
	/opt/etc/init.d/S80nginx start # added to start nginx on bootup
	{% endhighlight %}

15. If you give the command /opt/etc/init.d/S80nginx start right now, you may get the following error:
{% highlight js %}
	admin@RT-AC86U-52E0:/tmp/mnt/USBDISK/entware/etc/init.d# ./S80nginx start
	nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
	nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
	nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
	nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
	nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
	nginx: [emerg] still could not bind()
	{% endhighlight %}

	The fix is in RMerl's comment on this forum: https://github.com/RMerl/asuswrt-merlin.ng/issues/79
	The answer/fix is below:

		Asus does it on purpose for models with AiHome support, probably because the service relies on it. As a workaround, move httpd to a different port.
{% highlight js %}
		nvram set http_lanport=81
		nvram commit
		service restart_httpd	
		{% endhighlight %}	

16. Now if you do /opt/etc/init.d/S80nginx start:
{% highlight js %}
	admin@RT-AC86U-52E0:/tmp/mnt/USBDISK/entware/etc/init.d# ps | grep nginx
 	3546 admin    21244 S    nginx: master process nginx
 	3548 nobody   21244 S    nginx: worker process
 	3553 admin     5432 S    grep nginx
 	{% endhighlight %}

17. To make port 80 accessible from WAN, you must add port forwarding in the router for port 80:
    Go to WAN > Virtual Server/Port Forwarding > Port forwarding list
    Add the following configuration - 

    Service name: router_nginx

	{% highlight js %}
    External Port: 80
    Internal Port: 80
    Internal IP: 192.168.2.1
    Protocol: TCP
    {% endhighlight %}



