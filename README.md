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

Update:

keys are now auto-generated, the whole salt pillar is

so you have to run

```
vagrant up
```

After some churning the master and the 3 minions will be ready, but you have to:
```
vagrant ssh master
sudo salt 'minion*' state.highstate
```

I wanted to skip that, but the formula I got at https://github.com/jesusaurus/salt-formula-tinc doesn't take dependencies between the states in to account, so they aren't executed in the right order and the tinc deamon doesn't end up running. So for now issuing state.highstate is necessary.
But the whole formula is not suitable to be used in production anyway, e.g. all the nodes have access to the each others private keys

After everything's set up, you can go to a minion and see they can talk to eachother through the VPN.
the VPN addresses are respectively: 192.168.3.10, 192.168.3.11, 192.168.3.12
```
vagrant ssh minionN
ping 192.168.3.1*
nc -l 1234
nc 192.168.3.1* 1234
```
I had issues with VirtualBox not letting TCP through.

Warning:
the config uses pre-generated keys for the vagrant VMs - bad idea in production, but the goal here is to work on salt and tinc, so I didn't want to waste time writing the vagrant conf. Also the bridge interface name and the nodes' IP addresses are wired in, so depending on the environment might need to be changed.



salt '*' test.ping
salt '*' sys.doc
salt '*' cmd.run 'uname -a'

state.highstate
saltutil.sync_modules
saltutil.sync_all

salt '*' pillar.items
salt '*' saltutil.refresh_pillar

salt '*' grains.ls
salt '*' grains.items

salt-call -l debug state.sls tinc
salt-call state.sls tinc test=True

salt '*' state.highstate
salt '*' state.apply test