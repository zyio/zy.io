---
layout: post
title: "Hubot and Systemd"
quote: "Because everyone needs an IRC bot to send them pugs"
date: 2015-01-27T10:16:18+01:00
---

I've recently been building an IRC server for work, alongside other things, which finally gave us the chance to deploy an instance of `hubot`_, githubs automation/procrastination IRC bot.

It was also a good way to force myself to install a CentOS 7 box and get to grips with systemd, something I should have done a long time ago.

First major issue I hit was that hubot didn't have a unit file that I could find anywhere, this wasn't really an issue whilst I was testing, but when it went live I didn't want to have to rely on manually running a small bash wrapper I'd written or using an abomination like forever.js.

The bash wrapper I've written is a bit rough, but perfectly functional:

.. code-block:: bash
    
    #!/bin/bash
    GITLAB_CHANNEL="#announce" REDIS_URL="redis://127.0.0.1:6379/think" HUBOT_IRC_SERVER=irc.domain.tld  HUBOT_IRC_USESSL="true" HUBOT_IRC_SERVER_FAKE_SSL="true"  HUBOT_IRC_ROOMS="#general,#announce,#alerts"  HUBOT_IRC_NICK="hubot"   HUBOT_IRC_UNFLOOD="true"   HUBOT_IRC_PORT=6667   HUBOT_IRC_PASSWORD="HubotPasswordsAreTheBestPasswords" HUBOT_IRC_UNFLOOD="false" /home/hubot/bin/hubot -a irc --name hubot

To replace all that with a sensible unit file was quite a challenge, and I can't find evidence of anyone else having done it. Here's what I came up with:

.. code-block:: console

    [Unit]
    Description=Hubot is a robot
    After=ngircd.service

    [Service]
    Environment=GITLAB_CHANNEL=#announce
    Environment=HUBOT_IRC_SERVER=irc.domain.tld
    Environment=HUBOT_IRC_USESSL=true
    Environment=HUBOT_IRC_ROOMS=#general,#announce,#alerts
    Environment=HUBOT_IRC_NICK=hubot
    Environment=HUBOT_IRC_UNFLOOD=true
    Environment=HUBOT_IRC_PORT=6667
    Environment=HUBOT_IRC_PASSWORD=HubotPasswordsAreTheBestPasswords
    Environment=HUBOT_IRC_UNFLOOD=false
    Environment=PATH=node_modules/.bin:node_modules/hubot/node_modules/.bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
    WorkingDirectory=/home/hubot
    ExecStart=/usr/bin/node /home/hubot/node_modules/hubot/node_modules/.bin/coffee /home/hubot/node_modules/.bin/hubot -a irc --name hubot
    Restart=always
    StandardOutput=syslog
    SyslogIdentifier=hubot
    User=hubot
    Group=hubot


    [Install]
    WantedBy=multi-user.target


Reload systemd to put it all live:

``systemctl daemon-reload``


Then chkconfig it on (old habits die hard) and start it up:

``systemctl enable hubot``

``systemctl start hubot``


So far, that's about it really. It does what I want and it does it pretty well. It may be a bit rough in parts but it should be pretty transferable to your own install of hubot. Any comments/thoughts, please `get in touch.`_

.. _hubot: https://hubot.github.com/
.. _get in touch.: http://zy.io/about.html
