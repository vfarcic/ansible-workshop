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