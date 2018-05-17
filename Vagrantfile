# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

VAGRANTFILE_API_VERSION = "2"


settings = YAML.load_file 'global-config.yml'

$num_managers = settings['num_managers'] ? settings['num_managers'] : 1
$num_workers = settings['num_workers'] ? settings['num_workers'] : 2
$vm_cpus = settings['vm_cpus'] ? settings['vm_cpus'] : 1
$vm_mem = settings['vm_mem'] ? settings['vm_mem'] : 512
$first_manager_ip = settings['first_manager_ip'] ? settings['first_manager_ip'] : "198.18.1.2"
$ip_base = settings['ip_base'] ? settings['ip_base'] : "198.18.1."
$http_exposed_port = settings['http_exposed_port'] ? settings['http_exposed_port'] : 80

$num_nodes = $num_workers + $num_managers


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  def customize_vm(config)
    config.vm.box = "ubuntu/bionic64"

    config.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", $vm_mem]
      v.customize ["modifyvm", :id, "--cpus", $vm_cpus]

      v.customize ["modifyvm", :id, "--nictype1", "virtio"]
      v.customize ["modifyvm", :id, "--nictype2", "virtio"]
	  v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end

  $num_nodes.times do |n|
    node_name = "node-#{n+1}"
    config.vm.define "node-#{n+1}" do |node|
      customize_vm node
      node_ip = $ip_base + "#{n+2}"
      node.vm.network "private_network", ip: "#{node_ip}"
      if n < $num_managers
        node.vm.hostname = "manager-#{n+1}"
        # forward port only for the 1st manager
        # just because it's handy on single host
        # (my laptop)
        if n == 0
          node.vm.network "forwarded_port", guest: $http_exposed_port, host: 8080
        end
      else
        node.vm.hostname = "worker-#{n+1-$num_managers}"
      end
    # Only execute once the Ansible provisioner,
    # when all the machines are up and ready in
    # order to exploit ansible parallelism.
      if n == $num_nodes - 1
        node.vm.provision :ansible do |ansible|
          # Disable default limit to connect to all the machines
          ansible.limit = "all"
          ansible.playbook = "provision.yml"
          ansible.groups = {
            "first-manager" => ["node-1"],
            "managers" => ["node-[1:#{$num_managers}]"]
          }
          ansible.extra_vars = {
            "ansible_python_interpreter" => "/usr/bin/python3",
          }
        end
      end
    end
  end
end
