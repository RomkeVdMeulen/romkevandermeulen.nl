---
title: "Dokku and Docker on the same server: power and control"
date: 2015-01-31
tags:
  - docker
  - dokku
  - mysql
  - mariadb
  - nginx
  - ubuntu
  - wordpress
---
[Docker](https://www.docker.com) is a platform that allows you to deploy any
kind of app in a uniform way. [Dokku](http://progrium.viewdocs.io/dokku) builds
on this to create a full PaaS. It allows you to simply push your code repository
to your server and let Dokku build and deploy it for you, automatically.

While Dokku automatic builds are powerful and awesome, sometimes you want a bit
more control over how your app is deployed. For instance, you may want to deploy
your app in one container and link it with another container running your
database. Or you want to use one of the vast array of Docker images available in
the [Docker registry](https://registry.hub.docker.com/).

Dokku has a number of [community plugins](http://progrium.viewdocs.io/dokku/plugins#user-content-community-plugins "Dokku: community plugins")
that go a long way toward making such a scenario possible. Still, it doesn't
feel like the right tool for the job. After all, one of the greatest advantages
of Docker is that you can run similar or even identical containers on your
production server and on your development environment, ensuring painless
deploying. Dokku means giving up control of how your app will be deployed. Which
is great for simple applications or for testing, but for serious production use
you'll probably want that control back.

Let's say I have a modest number of projects I want to deploy, so I figure a
single server should be enough. I want to use pre-defined Docker images to
deploy my projects in production on this server. At the same time I also want to
have Dokku running on it so I can quickly push some new code to it to see if it
will work. It's taken some research to find how to effectively use Docker and
Dokku on a single server. Here's what you do.

## Setting up the server

I created a VPS on [DigitalOcean](https://www.digitalocean.com/?refcode=bd051b4d8067) 
using their pre-defined Dokku image. It turned out to be very up-to-date, with
the latest stable versions of Docker and Dokku running. If you're running your
own server, you can install the latest Docker
[like this]({{ site.baseurl }}/2015/01/29/getting-the-latest-version-of-docker-on-ubuntu.html "Getting the latest version of Docker on Ubuntu"),
and install Dokku using the
[install instructions](https://github.com/dokku/dokku#installing).

## Deploying a simple app with Dokku

After installing Dokku, you also need to specify on which domain you want Dokku
to deploy new apps. Write the domain name, e.g. `mydomain.com`, to
`/home/dokku/VHOST`. Now that Dokku is up and running, we need to register the
SSH key of our development machine so that we can push our code to Dokku. On
your development machine run this:

{% highlight shell %}
cat .ssh/id_rsa.pub| ssh root@mydomain.com sshcommand acl-add dokku myname
{% endhighlight %}

( The `myname` is to keep track of that key in case you want to delete it later. )

Now we're ready to use Dokku. To try it out, check out a sample
[node.js](http://nodejs.org/ "NodeJS") project on your development machine and
then push it to Dokku:

{% highlight shell %}
git clone git@github.com:heroku/node-js-sample.git
cd node-js-sample
git remote add dokku dokku@mydomain.com:test
git push dokku master
{% endhighlight %}

Now you should see something like this:

	Counting objects: 381, done.
	Delta compression using up to 4 threads.
	Compressing objects: 100% (308/308), done.
	Writing objects: 100% (381/381), 210.18 KiB | 0 bytes/s, done.
	Total 381 (delta 49), reused 373 (delta 44)
	-----> Cleaning up ...
	-----> Building test ...
	-----> Adding BUILD_ENV to build environment...
	-----> Node.js app detected
	-----> Requested node range: 0.10.x
	-----> Resolved node version: 0.10.35
	-----> Downloading and installing node
	-----> Exporting config vars to environment
	-----> Installing dependencies

	...

	-----> Running post-deploy
	-----> Creating new /home/dokku/test/VHOST...
	-----> Configuring test.mydomain.com...
	-----> Creating http nginx.conf
	-----> Running nginx-pre-reload
		   Reloading nginx
	=====> Application deployed:
		   http://test.mydomain.com

And, like it says right there, your application is now deployed to
`http://test.mydomain.com`!

## Deploying a Wordpress blog with Docker

Alright, so now we've seen how Dokku can build our apps for us. Now let's try
deploying a pre-existing Docker image: we'll deploy a blog using the
[official Wordpress Docker image](https://registry.hub.docker.com/_/wordpress/).
But before you can set up a new blog, you'll first need to set up a MySQL
database server. Here is how I did it:

{% highlight shell %}
docker run --name mysql --restart=always \
	   -e MYSQL_ROOT_PASSWORD=some-secret-string -d mariadb
{% endhighlight %}

The `--restart=always` will ensure that the Docker daemon starts up the container
again after an error or reboot. The Wordpress container can set up its own
database if you give it root access to the DB server, but I wanted to try doing
it myself. Rather than running a mysql client in another container, I installed
it directly on my server:

{% highlight shell %}
{% raw %}
apt-get install -qqy mysql-client
mysql -h`docker inspect --format "{{ .NetworkSettings.IPAddress }}" mysql` \
	  -uroot -p
{% endraw %}
{% endhighlight %}

The code between backticks retrieves the IP address assigned to my db server
container so I can connect to it.
\EDIT: [I've found a more elegant way to do this]({{ site.baseurl }}/2015/02/13/easy-access-dockerized-mysql-server.html "Easy access to a dockerized mysql server")
Now I can set up a database for my new blog by hand:

{% highlight mysql %}
CREATE DATABASE myblog;
CREATE USER 'myblog'@'%' IDENTIFIED BY 'another-password';
GRANT ALL ON myblog.* TO 'myblog'@'%';
FLUSH PRIVILEGES;
{% endhighlight %}

Now we're ready to deploy our Wordpress blog:

{% highlight shell %}
docker run --name myblog --link mysql:mysql \
	   -e WORDPRESS_DB_USER=myblog -e WORDPRESS_DB_PASSWORD=another-password \
	   -e WORDPRESS_DB_NAME=myblog -e VIRTUAL_HOST=blog.mydomain.com \
	   --restart=always -d wordpress
{% endhighlight %}

Now our blog container is running, as we can see by running `docker ps`.
However, we can only access its port `80` directly from the server. The
`VIRTUAL_HOST` env shows where we really want to access our new blog, but it's
not working yet.

## Reverse-proxy with nginx using docker-gen

We should already have [nginx](http://nginx.org/en/) running on our server since
Dokku depends on it. It wouldn't be hard to find the IP address of our blog
container and have nginx act as a reverse proxy for it, using our domain name.
The problem is that if our blog container is restarted, e.g. after a reboot of
our server, its IP address may have changed and we have to update our nginx
config by hand. Fortunately, somebody has been here before us and has created a
solution: [docker-gen](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/ "Automated Nginx Reverse Proxy for Docker").
It's a tool that automatically builds and updates config files for all running
containers. Let's install this wonderful tool:

{% highlight shell %}
cd /tmp
wget https://github.com/jwilder/docker-gen/releases/download/0.3.6/docker-gen-linux-amd64-0.3.6.tar.gz
tar xzf docker-gen-linux-amd64-0.3.6.tar.gz
mv docker-gen /etc/nginx/docker-gen
{% endhighlight %}

Now we have to create a template for the nginx config we want to generate. I
based mine on [jwilder's nginx-proxy](https://github.com/jwilder/nginx-proxy/blob/master/nginx.tmpl).
Write your config to `/etc/nginx/docker.template`. Now we can run docker-gen as
a single command. But if we want it to run automatically whenever our server
reboots, then we'll have to install it as a service. To do so, write this script
in `/etc/nginx/docker-gen-service`:

{% highlight bash %}
#!/bin/bash
/etc/nginx/docker-gen -only-exposed -watch -notify "service nginx reload" \
	  /etc/nginx/docker.template /etc/nginx/sites-enabled/docker_containers
{% endhighlight %}

And make it executable: `chmod +x /etc/nginx/docker-gen-service`. Now we'll
write some upstart config to `/etc/init/docker-nginx.conf`:

{% highlight conf %}
# docker-nginx - Nginx config generator for Docker containers

description "Nginx config generator for Docker containers"
author "Somebody <somebody@example.com>"

# When to start the service
start on filesystem and started docker

# When to stop the service
stop on runlevel [016]

# Automatically restart process if crashed
respawn

# Send output to logfile
console log

# Run before process
pre-start script
    [ -d /etc/nginx/certs ] || mkdir -p /etc/nginx/certs
end script

# Start the process
exec /etc/nginx/docker-gen-service
{% endhighlight %}

And now we start our new service: `initctl start docker-nginx`. Now docker-gen
is keeping an eye on all our Docker containers, and updating our nginx config to
match. To do this, it looks at the `VIRTUAL_HOST` env for each container.
Remember that for our Wordpress container we specified
`-e VIRTUAL_HOST=blog.mydomain.com` (if you want to assign more than one
hostname to a single container, you can use a comma-separated list:
`VIRTUAL_HOST=blog.a.com,test.b.com`). So provided our DNS settings are correct,
we should now be able to visit `http://blog.mydomain.com` and be greeted by the
Wordpress configuration screen.

* * *

So there you have it: you can now quickly and easily deploy an app using Dokku,
while at the same time deploying production apps more precisely with Docker.
