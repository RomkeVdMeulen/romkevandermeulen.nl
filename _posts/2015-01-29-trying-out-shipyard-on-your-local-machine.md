---
title: "Trying out Shipyard on your local machine"
date: 2015-01-29
tags:
  - docker
  - shipyard
---
**Note: this tutorial is outdated.**
Check [shipyard-project.com](http://shipyard-project.com/) for the latest
installation instructions.

[Shipyard](http://shipyard-project.com/) is a management console for
[Docker](https://www.docker.com/). You can use it to manage deployment of docker
images and containers on a number of different machines, but in this post I'm
going to show you how to try out shipyard on just your local machine. Be aware
that installing shipyard opens up a number of ports on your machine. Make sure
these aren't accessible to the public.

*   49153 - rethinkDB instance
*   49154 - rethinkDB cluster
*   49155 - rethinkDB web interface
*   4243 - Docker interface
*   8080 - Shipyard web interface

First make sure you have
[the latest version of Docker installed]({{ site.baseurl }}/2015/01/29/getting-the-latest-version-of-docker-on-ubuntu.html)
(the version in Ubuntu's repos is often too stale for shipyard). In this setup,
based on the [shipyard quickstart guide](http://shipyard-project.com/docs/quickstart/),
we'll be running shipyard inside a docker container. That means it can't access
the docker daemon on your machine directly. So we have make docker listen on
port `4243` so that shipyard can access it:

```bash
sudo sh -c "echo 'DOCKER_OPTS=\"-H tcp://:4243 -H unix:///var/run/docker.sock\"' >> /etc/default/docker"
```

Now we can run shipyard simply by pulling the necessary docker images from the
central index and running them:

```bash
docker run -it -d --name shipyard-rethinkdb-data \
	   --entrypoint /bin/bash shipyard/rethinkdb -l

docker run -it -P -d --name shipyard-rethinkdb \
	   --volumes-from shipyard-rethinkdb-data \
	   --restart=always shipyard/rethinkdb

docker run -it -p 8080:8080 -d --name shipyard \
	   --link shipyard-rethinkdb:rethinkdb \
	   --restart=always shipyard/shipyard
```

Your shipyard is now up and running. You can access the web interface at 
[http://localhost:8080](http://localhost:8080). We can also access shipyard by
CLI. For this we simply run another container. You might want to save this
command to a script or alias to make it easier to remember.

```bash
docker run --rm -it shipyard/shipyard-cli
```

Once you're on the CLI, you need to login to the shipyard instance we've just
setup. Remember: the CLI is running inside a separate container, with no direct
access to either your host machine or the shipyard server. To access either, we
need to use your host machine's public or local network IP address. You can find
the latter by running `ifconfig` and looking at the `inet addr` for your network
connection (usually `eth0`).

Run `shipyard login`. For host use the IP address you just found and port `8080`.
E.g.: [http://192.168.1.10:8080](http://192.168.1.10:8080/). The default user
is `admin` with password`shipyard`. Once you've logged into the CLI or Web
interface, we still need to connect our localhost docker daemon to shipyard
(remember that shipyard can't directly access your machine from inside its
container). In the web interface you can go to the engines tab. From the CLI you
can run this:

```bash
shipyard add-engine --id 'localhost' \
		 --addr 'http://[your-ip]:4243' \
		 --cpus '1.0' --memory '1024' \
		 --label 'local' --label 'dev'
```

Now that you have your shipyard set up and pointed at your local machine, you
should already be able to see several running containers, like shipyard itself.
You can add new containers by clicking “Deploy”. If you want, you can also
manage other servers running docker by adding them to the engines. But be sure
to secure your connection!
[This post]({{ site.baseurl }}/2015/01/30/setting-up-a-secure-public-connection-for-a-docker-daemon.html "Setting up a secure public connection for a Docker daemon")
will tell you how to expose the docker daemon securely.
