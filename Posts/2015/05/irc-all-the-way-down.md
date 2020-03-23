Permalink: /irc-all-the-way-down-znc-irccloud-quassel/
Title: IRC all the way down (ZNC + IRCCloud + Quassel)
Date: 2015-05-02
Tags: irc, tech

# IRC all the way down (ZNC + IRCCloud + Quassel)

For years, I felt that IRC was something I had to put up with. Most of the communities I want to be part of have a large IRC presence, and so I would fire up my trusty local IRC client, connect to [Freenode](https://freenode.net/) or [OFTC](https://www.oftc.net/), and try to learn from the excellent people who also hang out in various IRC communities. But I was always frustrated by the fact that I would miss discussions when I wasn't connected.

A few months back, a friend of mine introduced me to [Quassel](https://quassel-irc.org/), an open source software package that gets around IRC's major limitation (from my point of view): that your ability to read the contents of a channel are limited by your client being connected to the network. (The number of IRC loggers and other workarounds for persistence indicates others also find this a limitation.)

Quassel, in it's preferred configuration, requires at least two machines: a core that runs on an always-on server, and a client that connects to that core. The core is what actually connects to the IRC networks with your ident, and keeps a persistent connection for you. On the surface, this might not seem like an improvement over, say, irssi running on a server. It's an improvement for me because, despite several attempts, I have never been able to wrap my mind or fingers around irssi's keyboard shortcuts. Quassel has a nicer interface, a good desktop app, and some mobile mobile app support.

How do you get Quassel? Quite easily, if you're on an Ubuntu system. I recommend one of the cheap boxes from [DigitalOcean](https://www.digitalocean.com/?refcode=b86f40d85c65). They're easy to use, and only $5/month for a 512MB RAM / 20GB disk box.

On the server where you want your Quassel core to run, add the Quassel ppa to your apt repositories:

```shell
sudo add-apt-repository ppa:mamarley/quassel
```

Install the Quassel core package:

```shell
sudo apt-get update; sudo apt-get install quassel-core
```

You also want to make sure you've opened up port 4242 to outside traffic, as that's the port Quassel runs on. If you're not running a firewall (you probably should be!), you don't have to do anything. If you're running ufw like I am, you'll need to do this:

```shell
sudo ufw allow 4242
sudo ufw reload
```

Now that your core is all set up, let's configure it! One of the amazing things about Quassel is that you configure the core through the client. Download the client for your OS of choice, and it will walk you through how to get everything up and running.

So Quassel is great, and for a few months it served all my IRC needs perfectly well. But as I started getting more and more involved in communities on IRC, I started to feel the desire for a more mobile-ready solution. Quassel does have a free Android app, but I currently run iOS, and the iOS app didn't thrill me based on what I saw of it. I started looking for a better solution.

Some of my friends on IRC have been using [IRCCloud](https://www.irccloud.com/) for months, and they seemed to really enjoy it. I got an invite to the service from one of them, played around a bit, but didn't immediately see the appeal. At the time, I was still happy with my Quassel core and client. When I started hankering for a mobile solution, I gave IRCCloud another look, but didn't feel I could leave Quassel completely behind. By this point, I had given accounts on the core to some other friends interested in IRC, so I knew I couldn't shut it down. Plus, having Quassel as a backup in case IRCCloud ever went down seemed like a great idea. How could I get the best of both worlds, where Quassel and IRCCloud could use the same IRC connection, and I would never lose uptime?

Enter [ZNC](http://wiki.znc.in/ZNC). ZNC is an IRC bouncer, a piece of software that essentially proxies IRC connections for you. It connects to IRC, and you connect to it, similarly to Quassel. The difference is, the Quassel client speaks to the Quassel core over the Quassel protocol. You can connect to ZNC over IRC, using any client. Like IRCCloud, and the Quassel core.

How do you get setup with ZNC? On the same box where you're running that Quassel core, do:

```shell
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:teward/znc
sudo apt-get update
sudo apt-get install znc znc-dbg znc-dev znc-perl znc-python znc-tcl
```

This will add the ZNC ppa to your apt repositories, and install ZNC. Next you need to choose a user that will run the ZNC service. This could be your default user, although that's not recommended, and it most certainly shouldn't be the root user. I created a new user for running ZNC like this:
```shell
sudo adduser znc-admin
```

Before you configure ZNC to run under this user, you'll need to open another port in your firewall.

```shell
sudo ufw allow 5000
sudo ufw reload
```

Now you're ready to start up ZNC.

```shell
sudo su znc-admin
znc --makeconf
```

ZNC will ask you a whole bunch of questions, like what port to run on, what users to create, and how connections should be set up. The directions starting about halfway down [this DigitalOcean article](https://www.digitalocean.com/community/tutorials/how-to-install-znc-an-irc-bouncer-on-an-ubuntu-vps) are pretty good, and I followed most of their options, changing the user details to match what I needed. Once you've finished setup, ZNC will give you two important URLs: The URL to connect to the ZNC web interface, where you'll most likely configure ZNC going forward, and the URL for connecting an IRC client to ZNC. That connection URL will be in the form of:

```shell
{your server address or IP}:{port you chose} {username}:{password}
```

If you have an IRCCloud account, you'll need to pay special attention to those last bits, because {username}/{network name}:{password} will be your full server password to connect to the right account. For example:

```shell
UserName/freenode:password
```

When you add the network to IRCCloud, it'll look something like this:

[![IRCCLoud settings](https://wordfugue.s3.amazonaws.com/assets/wp-content/uploads/2015/05/Screen-Shot-2015-05-01-at-8.35.05-PM-300x250.png)](https://wordfugue.s3.amazonaws.com/assets/wp-content/uploads/2015/05/Screen-Shot-2015-05-01-at-8.35.05-PM.png)

You can use similar settings to connect Quassel to the same ZNC server.

Unfortunately, IRCCloud makes you upgrade your account to add servers with passwords. But in my opinion, IRCCloud is totally worth the $5/month. The more I use it, the more I like the service, the interface, and the mobile support. IRCCloud plus ZNC, with Quassel as a backup client connected to the same ZNC service, solves all my IRC woes. Hopefully, some combination of these services will be helpful to you as well.

And I'll see you on IRC.
