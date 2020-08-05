devstack-vagrant
================

**Forked from https://opendev.org/openstack/devstack-vagrant**

This is an attempt to build an easy to use tool to bring up a 2 node
devstack environment for local testing using Vagrant + Puppet.

It is *almost* fully generic, but still hard codes a few things about
my environment for lack of a way to figure out how to do this
completely generically (puppet templates currently hate me under
vagrant).

This will build a vagrant cluster that is L2 bridged to the interface
that you specify in ``config.yaml``. All devstack guests (2nd
level) will also be L2 bridged to that network as well. That means
that once you bring up this environment you will be able to ssh
stack@api (or whatever your hostname is) from any machines on your
network.

Vagrant Setup
------------------------

Install vagrant & virtual box

Configure a base ``~/.vagrant.d/Vagrantfile`` to set your VM size. If you
have enough horsepower you should make the file something like:

    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
        config.vm.provider :virtualbox do |vb|

             # Use VBoxManage to customize the VM. For example to change memory:
             vb.customize ["modifyvm", :id, "--memory", "5102"]
             vb.customize ["modifyvm", :id, "--cpus", "4"]
         end
    end

You can probably get away with less cpus, and 4096 MB of memory, but
the above is recommended size.

Local Setup
--------------------

```
$ vagrant plugin install vagrant-hostnamanager --local
$ vagrant plugin install vagrant-cachier --local
$ vagrant up 
```

Once done, login to the manager, and run : 

```
$ nova-manage cell_v2 discover_hosts --verbose
```

Add the following to you hosts file : 

```
10.0.10.10 manager.devstack.lab
10.0.10.20 compute.devstack.lab
```

Then go to **http://manager.devstack.lab**

What you should get
-----------------------------------
A 2 node devstack that includes cirros mini cloud image populated in glance.
You can get other images population such as fedora 20, ubuntu 12.04,
and ubuntu 14.04, just with a small addtion to ``extra_images`` part
in ``config.yaml.sample``.

Default security group with ssh and ping opened up.

Installation of the stack user ssh key as the default keypair.

Enable additional services
------------------------
The devstack environment created by this `Vagrantfile` includes just the basic
services to get started with OpenStack. If you want to try more services, you
can enable them on the manager node through the ``config.yaml`` file.

For example if you want to enable
[Swift](https://docs.openstack.org/developer/swift), you can add the
following line to your ``config.yaml``:

    manager_extra_services: s-proxy s-object s-container s-account
