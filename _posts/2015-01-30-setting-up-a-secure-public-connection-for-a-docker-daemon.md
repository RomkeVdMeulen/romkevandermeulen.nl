---
title: "Setting up a secure public connection for a Docker daemon"
date: 2015-01-30
tags:
  - docker
  - shipyard
  - nginx
---
I've posted about [how to set up shipyard on your local machine]({{ site.baseurl }}/2015/01/29/trying-out-shipyard-on-your-local-machine.html "Trying out Shipyard on your local machine").
You can also use your local shipyard to manage your remote servers, but to do
this you have to set up a secure connection to the Docker daemon on your server.
Docker has [an article](https://docs.docker.com/engine/security/https/) which tells you
how to do this. Since I've had to do it a couple of times over the last few
days, I figured I'd make a bash script for the process.

{% gist c04464b9730a7f01d27a %}

Once you've set this up, you can add your server's Docker daemon to shipyard in
the engine tab. Use the URL you passed to the script in the "Address" field with
port `4243`, e.g. `https://example.com:4243`. Copy the contents of 
`/etc/docker/cert.pem` to "SSL Certificate", `/etc/docker/key.pem` to "SSL Key"
and `/etc/docker/ca.pem` to "CA Certificate".
