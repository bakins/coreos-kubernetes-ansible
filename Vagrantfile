# -*- mode: ruby -*-

require 'yaml'

y = YAML.load_file("./group_vars/all")

$config = {
  channel: 'alpha',
  nodes: (ENV["NUM_NODES"] || 3).to_i,
  master_ip: y['kubernetes_master'] || "172.16.16.10"
}

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.vm.box = "coreos-%s" % $config[:channel]
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % $config[:channel]

  ["vmware_fusion", "vmware_workstation"].each do |vmware|
    config.vm.provider vmware do |v, override|
      override.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant_vmware_fusion.json" % $config[:channel]
    end
  end

  config.vm.provider :virtualbox do |v|
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end


  ansible_groups = {
    'master' => [ 'master'],
    'node' => [ ]
  }

  config.vm.define :master do |m|
    m.vm.hostname = "master"
    m.vm.network :private_network, ip: $config[:master_ip]
    config.vm.network "forwarded_port", guest: 8080, host: 8080

    m.vm.provision :file, :source => "user-data.yml", :destination => "/tmp/vagrantfile-user-data"
    m.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true

    m.vm.provision "ansible" do |ansible|
      ansible.playbook = "master.yml"
      ansible.groups = ansible_groups
    end
  end

  $config[:nodes].times do |i|
    config.vm.define "node-#{i}" do |n|
      ansible_groups['node'] << "node-#{i}"
      n.vm.hostname = "node-#{i}"
      n.vm.network :private_network, ip: "172.16.16.%d" % (100 + i)

      n.vm.provision :file, :source => "user-data.yml", :destination => "/tmp/vagrantfile-user-data"
      n.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true

      # HACK: need to make sure cloud-init runs for pypy
      n.vm.provision :shell, :inline => "until [ -f /home/core/.bootstrapped ]; do sleep 5; done"

      n.vm.provision "ansible" do |ansible|
        ansible.playbook = "node.yml"
        ansible.groups = ansible_groups
      end

    end
  end

end
