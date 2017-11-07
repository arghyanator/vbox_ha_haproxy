HAProxy cluster on VirtualBox and Vagrant 
=========================================

Spin up VirtualBox VM and install haproxy using Chef and Vagrant

Install Virtual Box on MAC:
---------------------------
http://download.virtualbox.org/virtualbox/5.1.30/VirtualBox-5.1.30-118389-OSX.dmg

Install Vagrant:
----------------
https://releases.hashicorp.com/vagrant/2.0.0/vagrant_2.0.0_x86_64.dmg


Install Ubuntu/Xenial 16.04 Virtual VM using Vagrant:
------------------------------------------------------
```
$ mkdir my_etcd
$ cd my_etcd
$ vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

$ vagrant box add ubuntu/xenial64
==> box: Loading metadata for box 'ubuntu/xenial64'
    box: URL: https://vagrantcloud.com/ubuntu/xenial64
==> box: Adding box 'ubuntu/xenial64' (v20171011.0.0) for provider: virtualbox
    box: Downloading: https://vagrantcloud.com/ubuntu/boxes/xenial64/versions/20171011.0.0/providers/virtualbox.box
==> box: Successfully added box 'ubuntu/xenial64' (v20171011.0.0) for 'virtualbox'!
```
Check if Vagrant Box was downloaded
```
$ vagrant box list
ubuntu/xenial64 (virtualbox, 20171011.0.0)
```

Modify Vagrantfile to add Chef Cookbooks (For 2 HA Node Haproxy):
------------------------------------------------------------------
```
$ egrep -v "^$|^#| #" Vagrantfile 
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
      
  
  config.vm.define :haproxy1 do |haproxy1|
    haproxy1.vm.box = "ubuntu/xenial64"
    haproxy1.vm.hostname = "haproxy1"
    haproxy1.vm.network :private_network, ip: "192.168.56.211"
    haproxy1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", "2048"]
      v.customize ["modifyvm", :id, "--cpus", "2"]
      v.customize ["modifyvm", :id, "--ioapic", "on"]
      v.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
      v.gui = true
    end
    haproxy1.vm.provision "shell", inline: "curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -v 13.2.20"
    haproxy1.vm.provision "chef_zero" do |chef|
      chef.cookbooks_path = "cookbooks"
      chef.data_bags_path = "data_bags"
      chef.nodes_path = "cookbooks/nodes"
  
      chef.add_recipe "install_haproxy_keepalived"
  
    end
  end
  config.vm.define :haproxy2 do |haproxy2|
    haproxy2.vm.box = "ubuntu/xenial64"
    haproxy2.vm.hostname = "haproxy2"
    haproxy2.vm.network :private_network, ip: "192.168.56.212"
    haproxy2.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", "2048"]
      v.customize ["modifyvm", :id, "--cpus", "2"]
      v.customize ["modifyvm", :id, "--ioapic", "on"]
      v.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
      v.gui = true
    end
    haproxy2.vm.provision "shell", inline: "curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -v 13.2.20"
    haproxy2.vm.provision "chef_zero" do |chef|
      chef.cookbooks_path = "cookbooks"
      chef.data_bags_path = "data_bags"
      chef.nodes_path = "cookbooks/nodes"
  
      chef.add_recipe "install_haproxy_keepalived"
  
    end
  end
end

end

```
 
Boot up 2 VMs and Install haproxy and keepalived in it using Vagrant
--------------------------------------------------------------------
```
$ vagrant up haproxy1 haproxy2
```

Chef configuration
------------------
```
$ tree cookbooks/
cookbooks/
├── install_haproxy_keepalived
│   ├── files
│   │   ├── haproxy
│   │   └── haproxy_1.7.9-1ubuntu0.1_amd64.deb
│   ├── recipes
│   │   ├── default.rb
│   │   └── install_haproxy.rb
│   └── templates
│       └── keepalived.erb
└── nodes
    ├── haproxy1.json
    └── haproxy2.json
````
haproxy setup
-------------
```
ubuntu@haproxy1:~$ ps -eaf |grep haproxy
root      3805     1  0 04:52 ?        00:00:00 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
haproxy   3807  3805  0 04:52 ?        00:00:00 /usr/sbin/haproxy-master
haproxy   3808  3807  0 04:52 ?        00:00:00 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
```

```
ubuntu@haproxy2:~$ ps -eaf |grep haproxy
root      3801     1  0 04:54 ?        00:00:00 /usr/sbin/haproxy-systemd-wrapper -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid
haproxy   3802  3801  0 04:54 ?        00:00:00 /usr/sbin/haproxy-master
haproxy   3804  3802  0 04:54 ?        00:00:00 /usr/sbin/haproxy -f /etc/haproxy/haproxy.cfg -p /run/haproxy.pid -Ds
```

Keepalived status
-----------------
Log in to first Haproxy node and check VIP 
```
ubuntu@haproxy1:~$ sudo su -
root@haproxy1:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 02:51:17:41:23:cb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::51:17ff:fe41:23cb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:52:a1:e9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.211/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet 192.168.56.220/32 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe52:a1e9/64 scope link 
       valid_lft forever preferred_lft forever
```
Note: The line shows the VIP configured on primary node - __inet 192.168.56.220/32 scope global enp0s8__

```
ubuntu@haproxy1:~$ grep Keepalived_vrrp /var/log/syslog
Nov  7 04:52:42 ubuntu-xenial Keepalived_vrrp[3826]: Registering Kernel netlink reflector
Nov  7 04:52:42 ubuntu-xenial Keepalived_vrrp[3826]: Registering Kernel netlink command channel
Nov  7 04:52:42 ubuntu-xenial Keepalived_vrrp[3826]: Registering gratuitous ARP shared channel
Nov  7 04:52:42 ubuntu-xenial Keepalived_vrrp[3826]: Opening file '/etc/keepalived/keepalived.conf'.
Nov  7 04:52:42 ubuntu-xenial Keepalived_vrrp[3826]: Configuration is using : 66096 Bytes
Nov  7 04:52:42 ubuntu-xenial Keepalived_vrrp[3826]: Using LinkWatch kernel netlink reflector...
Nov  7 04:52:42 ubuntu-xenial Keepalived_vrrp[3826]: VRRP_Script(haproxy) succeeded
Nov  7 04:52:43 ubuntu-xenial Keepalived_vrrp[3826]: VRRP_Instance(52) Transition to MASTER STATE
Nov  7 04:52:44 ubuntu-xenial Keepalived_vrrp[3826]: VRRP_Instance(52) Entering MASTER STATE
Nov  7 04:54:09 ubuntu-xenial Keepalived_vrrp[3826]: VRRP_Instance(52) Received lower prio advert, forcing new election
Nov  7 04:54:09 ubuntu-xenial Keepalived_vrrp[3826]: VRRP_Instance(52) Received lower prio advert, forcing new election
```

```
ubuntu@haproxy2:~$ grep Keepalived_vrrp /var/log/syslog
Nov  7 04:54:08 ubuntu-xenial Keepalived_vrrp[3821]: Registering Kernel netlink reflector
Nov  7 04:54:08 ubuntu-xenial Keepalived_vrrp[3821]: Registering Kernel netlink command channel
Nov  7 04:54:08 ubuntu-xenial Keepalived_vrrp[3821]: Registering gratuitous ARP shared channel
Nov  7 04:54:08 ubuntu-xenial Keepalived_vrrp[3821]: Opening file '/etc/keepalived/keepalived.conf'.
Nov  7 04:54:08 ubuntu-xenial Keepalived_vrrp[3821]: Configuration is using : 66096 Bytes
Nov  7 04:54:08 ubuntu-xenial Keepalived_vrrp[3821]: Using LinkWatch kernel netlink reflector...
Nov  7 04:54:08 ubuntu-xenial Keepalived_vrrp[3821]: VRRP_Script(haproxy) succeeded
Nov  7 04:54:09 ubuntu-xenial Keepalived_vrrp[3821]: VRRP_Instance(52) Transition to MASTER STATE
Nov  7 04:54:09 ubuntu-xenial Keepalived_vrrp[3821]: VRRP_Instance(52) Received higher prio advert
Nov  7 04:54:09 ubuntu-xenial Keepalived_vrrp[3821]: VRRP_Instance(52) Entering BACKUP STATE
```

Destroy VirtualBox VM
---------------------
```
$ vagrant destroy -f
```
