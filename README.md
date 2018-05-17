# Docker Swarm cluster provisione with Ansible and Vagrant 


## Requirements

* Vagrant, version 2.x
    * https://www.vagrantup.com/docs/installation/
* Virtualbox
    * https://www.virtualbox.org/wiki/Downloads
* Ansible, version 2.5.x or higher
    * https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html


## Setup Infrastructure

By default it's going to build a 3 node swarm cluster. Three manager and two workers, those numbers can be changed in the 'global-config.yml'.
In order to have a cluster that's resilient to faults and crashes you need to have redundancy on the masters. An odd number(>=3) of masters is needed and suggested. The actual amount depends on the number of faults you want to tolerate.

    vagrant up

