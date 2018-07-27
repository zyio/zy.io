---
layout: post
title: "Internal > External IP mapping"
tags:
- oneliners
- linux
date: 2015-02-22T01:17:03+01:00
---

In a hosting environment, it's not too uncommon to find yourself with a server full of IP addresses starting 172 or 192. These IPs are private range and are typically translated to 'real' public IPs by the firewall or other network device in front of your server.

That's all well and good, but when you need to find your external IP, you're stuck consulting solution documents or curling icanhazip.com.

Below is a small command I cobbled together to iterate over all the internal IPs on your server and print out their external mapping:

.. code-block:: bash

    for i in $(ip a|grep -E '172.|192.'|sort -t . -k 3,3n -k 4,4n|awk '{print $2}'|sed "s/\/.*//"); do echo "$i <-> `curl -s --interface $i icanhazip.com`";done


