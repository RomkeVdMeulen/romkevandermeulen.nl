---
title: "Easy access to a dockerized mysql server"
date: 2015-02-13
excerpt:
  Deploying a new mysql server using Docker is quite easy. But once the server
  is up, you want root access to it from your command-line. Here's a little
  shell function that'll do it for you.
tags:
  - docker
  - shell
  - mysql
  - mariadb
---
Deploying a new mysql server using Docker is quite easy:

{% highlight shell %}
docker run --name mysql mariadb
{% endhighlight %}

But once the server is up, you want root access to it from your command-line.
The [mariadb repo](https://registry.hub.docker.com/_/mariadb/) shows an example
of how to do this. I've wrapped their command in a little shell function that
you can add to your `.bashrc` or `.zshrc`. If you call the function without
arguments, it will drop you on the mysql prompt. If you give a mysql command as
argument, it will execute it as mysql root and show the result.

{% gist b84d07b8f3f7bfd73f03 %}

Here's how it works:

    # mysqlroot "SHOW DATABASES"
    Database
    my_db_1
    my_db_2

    # mysqlroot
    MariaDB [(none)]>

## Bonus round: automated backup

{% gist a74abb55b1c246474a4b %}
