# Salt, Tinc, Vagrant

Vagrant setup for 3 node tinc VPN.

Both Vagrant and Salt can be used to deploy nodes to the cloud. Here vagrant is used to configure four local VMs, just as if they were created manually, but without having to go into each one to install Salt and other stuff, and configure it.

You can get vagrant at https://www.vagrantup.com/downloads.html

after you get it

```
git clone https://github.com/klnvnv/salt-tinc-vagrant.git

cd salt-tinc-vagrant

vagrant up

```

It will bring up one master and three minions - 4 VMs altogether, so it'll take some time to do that - about half an hour.

After it's done you'll have 4 nodes - master, minion0, minion1, minion2.

```
vagrant ssh master

```
or

```
vagrant ssh minionN

```

to ssh into the nodes and take a look around

Warning:
the config uses pre-generated keys for the vagrant VMs - bad idea in production, but the goal here is to work on salt and tinc, so I didn't want to waste time writing the vagrant conf. Also the bridge interface name and the nodes' IP addresses are wired in, so depending on the environment might need to be changed.

NOT DONE YET
I just got up to the point where I installed tinc on the minions,
but VPN between them is not configured yet.