---
title: "Running the Glassfish pkg tool on 64 bit Ubuntu"
date: 2016-09-03
tags:
  - glassfish
  - linux
  - ubuntu
---
The Glassfish Java application server comes with a number of tools, including
`pkg`. Unfortunately, `pkg` is a 32 bit Python application and almost all of us
run 64 bit systems. So you need a couple of compatibility libraries to run the
tool. Here's how you do that on Ubuntu 16.04.

{% gist 0ed35627b9fa81693903bab85a8b7835 %}