==============
EC2 and Django
==============


My configuration for micro ec2 django instances that will be serving mostly static content


Choosing EC2 AMI
----------------

I went with the latest Ubuntu Hardy image.  I've used it in the past and
it works fine::

    ami-cb8d61a2

Amazon portal config:

1. Choose the image and click *launch*  
2. Go through the wizard, choose *micro*
instance and go through the defaults until you set up your private key.
3. Choose something meaningful to the server you're building.  Create the
private keypair (a .pem file), download it, and move it to your **~/.ssh** directory.
4. For firewall, enable ssh and http access. I usually give it a group name like 'quick-start'
4a. For larger apps, you might want to enable more than just ssh and http,
like mysql access, or smtp for mail service, etc.
5. Hit launch


Installing utils
----------------

We're ready to start installing things

::
    
    Note that if you just made your instance, you'll have to wait several
    minutes for Amazon to boot it up and hand it over for ssh access


ssh into your brand new ubuntu box on ec2::

    ssh -i ~/.ssh/myprivatefile.pem ubuntu@ec2-##-##-##-###.compute-1.amazonaws.com


If you get a permission error, like this:

::

    Permissionssions 0644 for '/Users/buf/.ssh/myprivatefile.pem' are too open.
    It is recommended that your private key files are NOT accessible by others.
    This private key will be ignored.
    bad permissions: ignore key: /Users/buf/.ssh/myprivatefile.pem
    Permission denied (publickey).m 0644 for '/Users/buf/.ssh/myprivatefile.pem' are too open.

I fixed this with writing the pem rules to 600::

    chmod 600 ~/.ssh/myprivatefile.pem

After we've ssh'd in.  Let's install django, python mysqldb, mysql server, and mod_wsgi::

    sudo aptitude install python-django
    sudo aptitude install mysql-server
    sudo aptitude install python-mysqldb
    sudo aptitude install libapache2-mod-wsgi

Setting up wsgi
---------------

Setting up bare bones django::

    cd /usr/local
    mkdir www
    cd www
    django-admin startproject myproject


Make a new sites-available for your new django project::

    sudo vim /etc/apache2/sites-available/myproject.com


Write standard sites config::

    <VirtualHost *:80>
        ServerName myproject.com
        ServerAlias www.myproject.com
        WSGIScriptAlias / /usr/local/www/myproject/myproject.wsgi
    </VirtualHost>



Make myproject.wsgi file::
    
    vim /usr/locale/www/myproject/myproject.wsgi

Write standard myproject.wsgi file::

    import os
    import sys

    sys.path.append('/usr/local/www/myproject/')

    os.environ['DJANGO_SETTINGS_MODULE'] = 'myproject.settings'

    import django.core.handlers.wsgi

    application = django.core.handlers.wsgi.WSGIHandler()

Start it up
-----------

Enable the site in apache and reload::

    sudo a2ensite mysite.com
    sudo /etc/init.d/apache2 reload

 
