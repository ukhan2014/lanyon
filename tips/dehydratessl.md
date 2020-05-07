---
layout: note
title:  How to use dehydrated to procure certificates using DNS challenge
---

{% highlight js %}
root@beardom:~/dehydrated/dehydrated-master# cat config 
WELLKNOWN="/var/www/html/.well-known/acme-challenge"
# Path to certificate authority (default: https://acme-v02.api.letsencrypt.org/directory)
#CA="https://acme-staging-v02.api.letsencrypt.org/directory"

root@beardom:~/dehydrated/dehydrated-master# vi config 
root@beardom:~/dehydrated/dehydrated-master# ./dehydrated --domain nc.beardom.xyz --cron
# INFO: Using main config file /root/dehydrated/dehydrated-master/config

To use dehydrated with this certificate authority you have to agree to their terms of service which you can find here: https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf

To accept these terms of service run `./dehydrated --register --accept-terms`.
root@beardom:~/dehydrated/dehydrated-master# ./dehydrated --register --accept-terms
# INFO: Using main config file /root/dehydrated/dehydrated-master/config
+ Generating account key...
+ Registering account key with ACME server...
+ Fetching account URL...
+ Done!
root@beardom:~/dehydrated/dehydrated-master# ./dehydrated --domain nc.beardom.xyz --cron
# INFO: Using main config file /root/dehydrated/dehydrated-master/config
Processing nc.beardom.xyz
 + Creating new directory /root/dehydrated/dehydrated-master/certs/nc.beardom.xyz ...
 + Signing domains...
 + Generating private key...
 + Generating signing request...
 + Requesting new certificate order from CA...
 + Received 1 authorizations URLs from the CA
 + Handling authorization for nc.beardom.xyz
 + 1 pending challenge(s)
 + Deploying challenge tokens...
 + Responding to challenge for nc.beardom.xyz authorization...
 + Challenge is valid!
 + Cleaning challenge tokens...
 + Requesting certificate...
 + Checking certificate...
 + Done!
 + Creating fullchain.pem...
 + Done!
root@beardom:~/dehydrated/dehydrated-master# cd certs/
root@beardom:~/dehydrated/dehydrated-master/certs# ls
nc.beardom.xyz
root@beardom:~/dehydrated/dehydrated-master/certs# cd nc.beardom.xyz/
root@beardom:~/dehydrated/dehydrated-master/certs/nc.beardom.xyz# ls
cert-1588415561.csr  cert.csr  chain-1588415561.pem  fullchain-1588415561.pem  privkey-1588415561.pem
cert-1588415561.pem  cert.pem  chain.pem             fullchain.pem             privkey.pem
root@beardom:~/dehydrated/dehydrated-master/certs/nc.beardom.xyz# ls -alh
total 28K
drwx------ 2 root root 4.0K May  2 03:32 .
drwx------ 3 root root 4.0K May  2 03:32 ..
-rw------- 1 root root 1.7K May  2 03:32 cert-1588415561.csr
-rw------- 1 root root 2.2K May  2 03:32 cert-1588415561.pem
lrwxrwxrwx 1 root root   19 May  2 03:32 cert.csr -> cert-1588415561.csr
lrwxrwxrwx 1 root root   19 May  2 03:32 cert.pem -> cert-1588415561.pem
-rw------- 1 root root 1.7K May  2 03:32 chain-1588415561.pem
lrwxrwxrwx 1 root root   20 May  2 03:32 chain.pem -> chain-1588415561.pem
-rw------- 1 root root 3.9K May  2 03:32 fullchain-1588415561.pem
lrwxrwxrwx 1 root root   24 May  2 03:32 fullchain.pem -> fullchain-1588415561.pem
-rw------- 1 root root 3.2K May  2 03:32 privkey-1588415561.pem
lrwxrwxrwx 1 root root   22 May  2 03:32 privkey.pem -> privkey-1588415561.pem
root@beardom:~/dehydrated/dehydrated-master/certs/nc.beardom.xyz#
{% endhighlight %}

