## CoSciN On a Laptop

This setup emulates the CoSciN network on an Ubuntu laptop.  Technical background and details are at
http://coscin.github.io/website/laptop.html.

To install, start with an Ubuntu-based host.  We've tested on an Ubuntu 14.04 desktop setup with 8 GB RAM.
The host must have a _wired_ ethernet coonnection to the Internet.  User-mode networking in KVM will not allow VM's
to bridge to a wireless ethernet conneciton.  

Then:

```
$ git clone http://github.com/coscin/laptop
```

All of the KVM images will use the same username `ubuntu` with the password preconfigured in the `laptop/install`
script.  You may want to change it to suit your purposes.

The following will run a long time because it grabs the Ubuntu Cloud Server 15.10 image from the Internet (which is about 300 MB).

```
$ cd laptop
$ ./install
$ cd ..
$ source .profile
$ newimg ithaca
$ newimg nyc
$ newimg router
$ newimg bastion
```

The host VM's `ithaca` and `nyc` normally access the private network you're building, but not the
Internet.  Fortunately, the VM's have a dormant network interface "eth1" that provides a bridge to user-mode
networking and an Ethernet connection.  You merely need to bring eth1 up, install software or whatever you need to do
with the Internet connection, then bring it back down (which
is important because CoSciN traffic must use the tap as a default route.)

To do this on Ithaca:

```
$ sudo bin/ithaca_up
```

Then login as ubuntu, and you'll be running the VM directly in your terminal window.

```
ubuntu@ithaca$ sudo hostnamectl set-hostname coscintest-host-ithaca
sudo: unable to resolve host cosscintest-ctrl-ithaca
ubuntu@ithaca$ exit

Ubuntu 15.10 coscintest-host-ithaca ttyS0

coscintest-host-ithaca login: ubuntu
Password: *******

ubuntu@ithaca$ sudo nano /etc/network/interfaces.d/eth0.cfg
```

`eth0.cfg` will look like this:

```
# The primary network interface
auto eth0
iface eth0 inet static
    address 192.168.56.100/24
    gateway 192.168.56.1
```

And eth1.cfg (same directory) will look like:

```
iface eth1 inet dhcp
  up ip route replace default via 10.0.2.2
  down ip route replace default via 192.168.56.1
```    

You can test the Internet connection if you wish by bringing up eth1:

```
ubuntu@ithaca$ sudo ifup eth1
ubuntu@ithaca$ sudo apt-get update
```

The apt-get ensures your Internet connection is good (unfortunately, Ping doesn't work across KVM usermode networking, so don't
even try it.)   Before you start testing, however, you must
shut down the Internet connection:

```
ubuntu@ithaca$ sudo ifdown eth1
```

