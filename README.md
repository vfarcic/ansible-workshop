Prerequisites
=============

Install [Virtual Box](https://www.virtualbox.org/), [Vagrant](https://www.vagrantup.com/) and [Git](https://git-scm.com/).

__If you are using Windows, please make sure that Git is configured to use "Checkout as-is". This can be accomplished during the setup by selecting the second or third option from the screen depicted in the Figure 4-1.__

TODO: Git clone

__Vagrant tends to create problems with file permissions when linux VMs are created from a Windows host. If you are a Windows user, please follow the following instructions. After cloning the code, open the Vagrantfile in your text editor (I tend to use [NotePad++](https://notepad-plus-plus.org/)). Inside, you'll find the following two lines:__

```
  config.vm.synced_folder ".", "/vagrant"
  # config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=700,fmode=600"]
```

__Please comment the first and uncomment the second line. The end result should be as follows.__

```
  # config.vm.synced_folder ".", "/vagrant"
  config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=700,fmode=600"]
```

Create Environments
===================

```bash
cd ansible-workshop

cat Vagrantfile

vagrant up

vagrant ssh cd

cat /vagrant/ansible/hosts/prod

cat /vagrant/ansible/swarm.yml
```

Common Role
===========

Tasks
-----

* Install __jq__ package
* Run the Ansible Playbook

Help
----

* [APT Module](http://docs.ansible.com/ansible/apt_module.html)
* Run the Ansible Playbook

```bash
ansible-playbook /vagrant/ansible/swarm.yml \
    -i /vagrant/ansible/hosts/prod
```

Test
----

```bash
curl 'https://api.github.com/repos/vfarcic/ansible-workshop/commits?per_page=5' | jq '.'
```

Consul Role
===========

Tasks
-----

* Create the following directories:

  * /data/consul/logs
  * /data/consul/data
  * /data/consul/config

* Copy the following files:

  * consul > /usr/local/bin/consul (mode 0755)
  * ui > /data/consul (mode 0644)

* Refactor to use variables
* Run the following command

```bash
nohup consul agent [EXTRA] \
    -ui-dir /data/consul/ui \
    -data-dir /data/consul/data \
    -config-dir /data/consul/config \
    -node=[HOST_NAME] \
    -bind=[IP] \
    -client=0.0.0.0 \
    >/data/consul/logs/consul.log 2>&1 &
```

  * __[EXTRA]__ should be replaced with `-server -bootstrap-expect 1` on the __main server__ and be empty on all others.
  * __[HOST_NAME]__ should be replaced with the name of the host.
  * __[IP]__ should be replaced with the IP of the host

* Run the following command only if not the __main server__ (10.100.192.200).

```bash
consul join [IP_OF_THE_MAIN_SERVER]
```

  * __[IP_OF_THE_MAIN_SERVER]__ should be replaced with the IP of the main server (10.100.192.200)

Help
----

* [File Module](http://docs.ansible.com/ansible/file_module.html)
* [Copy Module](http://docs.ansible.com/ansible/copy_module.html)
* All files are already available in roles/consul/file
* [Shell Module](http://docs.ansible.com/ansible/shell_module.html)
* [Loops](http://docs.ansible.com/ansible/playbooks_loops.html)
* [Inventory](http://docs.ansible.com/ansible/intro_inventory.html)
* __ansible_hostname__ provides host name.
* __facter_ipaddress_eth1__ provides host IP.
* [Conditionals](http://docs.ansible.com/ansible/playbooks_conditionals.html)

Test
----

```bash
curl 10.100.192.200:8500/v1/catalog/nodes \
	| jq '.'
```

Open [http://10.100.192.200:8500/](http://10.100.192.200:8500/)

Docker
======

Tasks
-----

* Add Docker repository __deb https://apt.dockerproject.org/repo ubuntu-trusty main__ and update cache.
* Install __docker-engine__ and __python-pip__ Debian packages
* Install __docker-py__ PIP package version __0.4.0__

Help
----

* [APT Repository Module](http://docs.ansible.com/ansible/apt_repository_module.html)
* [APT Module](http://docs.ansible.com/ansible/apt_module.html)
* [PIP Module](http://docs.ansible.com/ansible/pip_module.html)

Test
----

```bash
sudo docker images
```

Swarm
=====

Tasks
-----

* Copy configuration file __docker.cfg__ to __/etc/default/docker__ (only if NOT main server)
* Restart the __docker__ service if the configuration file __/etc/default/docker__ was changed
* Run Docker container called __swarm__

  * If main server, command should be `manage consul://[IP]:8500/swarm`
  * If NOT main server, command should be `join --advertise=[IP]:2375 consul://[IP]:8500/swarm`
  * Replace [IP] with the IP of the server

Help
----

* [Copy Module](http://docs.ansible.com/ansible/copy_module.html)
* [Variables: Registered Variables](http://docs.ansible.com/ansible/playbooks_variables.html#registered-variables)
* [Service Module](http://docs.ansible.com/ansible/service_module.html)
* [Docker Module](http://docs.ansible.com/ansible/docker_module.html)

Test
----

```bash
export DOCKER_HOST=tcp://10.100.192.200:2375

docker info

docker run -d --name hello-world \
    ubuntu /bin/sh -c "while true; do echo hello world; sleep 5; done"

docker ps -a

docker logs -f hello-world
```
