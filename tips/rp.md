---
layout: note
title:  Resize partition to fill up hdd on AWS EC2 Instance
---

1. First do df -h to see your disk usage:
{% highlight js %}
 ubuntu@ip-172-31-41-76:~/nougat$ df -h
 Filesystem      Size  Used Avail Use% Mounted on
 udev             16G     0   16G   0% /dev
 tmpfs           3.2G  8.7M  3.1G   1% /run
 /dev/nvme0n1p1   97G   94G  3.8G  97% /               <<<
 tmpfs            16G     0   16G   0% /dev/shm
 tmpfs           5.0M     0  5.0M   0% /run/lock
 tmpfs            16G     0   16G   0% /sys/fs/cgroup
 /dev/loop0       94M   94M     0 100% /snap/core/8935
 /dev/loop1       18M   18M     0 100% /snap/amazon-ssm-agent/1566
 /dev/loop2       94M   94M     0 100% /snap/core/9066
 tmpfs           3.2G     0  3.2G   0% /run/user/1000

{% endhighlight %}

	 Above we can see that the root partition only has 97G space. This
	 VM has 500G hard drive.

2. Then do sudo lsblk to see what is going on with the partitions:

{% highlight js %}
 ubuntu@ip-172-31-34-208:~/nougat$ lsblk
 NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
 loop0         7:0    0   18M  1 loop /snap/amazon-ssm-agent/1566
 loop1         7:1    0 93.8M  1 loop /snap/core/8935
 loop2         7:2    0 93.9M  1 loop /snap/core/9066
 nvme0n1     259:0    0  500G  0 disk 
 └─nvme0n1p1 259:1    0   97G  0 part /                <<<

{% endhighlight %}

 With lsblk we can see that the disk is in fact 500G but the partition is
 only 97G. We need to expand the partition first.

3. Make sure growpart is installed. On Ubuntu this is in the cloud-utils pkg.
{% highlight js %}
 sudo apt-get install cloud-utils
{% endhighlight %}

4. Grow the partition:
{% highlight js %}
 sudo growpart /dev/nvme0n1 1
{% endhighlight %}

5. Grow the filesystem:
{% highlight js %}
 sudo resize2fs /dev/nvme0n1p1
{% endhighlight %}

6. Do df -h again:
{% highlight js %}
 ubuntu@ip-172-31-34-208:~/nougat$ df -h
 Filesystem      Size  Used Avail Use% Mounted on
 udev             16G     0   16G   0% /dev
 tmpfs           3.2G  8.7M  3.1G   1% /run
 /dev/nvme0n1p1  485G   95G  391G  20% /                <<<
 tmpfs            16G     0   16G   0% /dev/shm
 tmpfs           5.0M     0  5.0M   0% /run/lock
 tmpfs            16G     0   16G   0% /sys/fs/cgroup
 /dev/loop1       94M   94M     0 100% /snap/core/8935
 /dev/loop0       18M   18M     0 100% /snap/amazon-ssm-agent/1566
 /dev/loop2       94M   94M     0 100% /snap/core/9066
 tmpfs           3.2G     0  3.2G   0% /run/user/1000	

{% endhighlight %}

 Now we can see 485G available on the root partition

7. do sudo lsblk again:
{% highlight js %}
 ubuntu@ip-172-31-34-208:~/nougat$ lsblk
 NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
 loop0         7:0    0   18M  1 loop /snap/amazon-ssm-agent/1566
 loop1         7:1    0 93.8M  1 loop /snap/core/8935
 loop2         7:2    0 93.9M  1 loop /snap/core/9066
 nvme0n1     259:0    0  500G  0 disk 
 └─nvme0n1p1 259:1    0  500G  0 part /               <<<

{% endhighlight %}

 We can see the partition is making full use of the disk. 
