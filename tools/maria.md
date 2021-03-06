---
layout: note
title:  Installing mariadb in a docker container
---

First start the mysql service:

{% highlight js %}
root@094fcef21010:~# service mysql status                                                                                       
 * MariaDB is stopped.                                                                                                          
root@094fcef21010:~# service mysql start                                                                                        
 * Starting MariaDB database server mysqld                                                                               [ OK ] 
{% endhighlight %}

Now give the following command to set the root password:
{% highlight js %}
root@094fcef21010:~# mysql_secure_installation                                                                                  
                                                                                                                                
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB                                                           
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!                                                              
                                                                                                                                
In order to log into MariaDB to secure it, we'll need the current                                                               
password for the root user.  If you've just installed MariaDB, and                                                              
you haven't set the root password yet, the password will be blank,                                                              
so you should just press enter here.                                                                                            
                                                                                                                                
Enter current password for root (enter for none):                                                                               
OK, successfully used password, moving on...                                                                                    
                                                                                                                                
Setting the root password ensures that nobody can log into the MariaDB                                                          
root user without the proper authorisation.                                                                                     
                                                                                                                                
Set root password? [Y/n] Y                                                                                                      
New password:                                                                                                                   
Re-enter new password:                                                                                                          
Sorry, passwords do not match.                                                                                                  
                                                                                                                                
New password:                                                                                                                   
Re-enter new password:                                                                                                          
Sorry, you can't use an empty password here.                                                                                    
                                                                                                                                
New password:                                                                                                                   
Re-enter new password:                                                                                                          
Password updated successfully!                                                                                                  
Reloading privilege tables..                                                                                                    
 ... Success!                                                                                                                       [13/1506]

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
{% endhighlight %}

Now you can log in as root:

{% highlight js %}
root@094fcef21010:~# mysql -uroot -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 57
Server version: 10.3.22-MariaDB-1ubuntu1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit
Bye

{% endhighlight %}


