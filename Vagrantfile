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
    config.vm.network :private_network, ip: $config[:master_ip]

    config.vm.provision :file, :source => "user-data.yml", :destination => "/tmp/vagrantfile-user-data"
    config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "master.yml"
      ansible.groups = ansible_groups
    end
  end

  $config[:nodes].times do |i|
    config.vm.define "node-#{i}" do |n|
      ansible_groups['node'] << "node-#{i}"
      n.vm.hostname = "node-#{i}"
      config.vm.network :private_network, ip: "172.16.16.%d" % (100 + i)

      config.vm.provision :file, :source => "user-data.yml", :destination => "/tmp/vagrantfile-user-data"
      config.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true

      config.vm.provision "ansible" do |ansible|
        ansible.playbook = "node.yml"
        ansible.groups = ansible_groups
      end

    end
  end

end
