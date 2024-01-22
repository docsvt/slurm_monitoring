# -*- mode: ruby -*-
# vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = "2"

require 'yaml'

current_dir = File.dirname(File.expand_path(__FILE__))
DEPLOYMENT_CONFIG_PATH = File.join(current_dir, ".deployment/slurm_monitoring.yaml")

# Load settings from DEPLOYMENT_CONFIG_PATH or deployment.yml.dist

if File.file?("#{DEPLOYMENT_CONFIG_PATH}")
    deployment = YAML.load_file("#{DEPLOYMENT_CONFIG_PATH}")
else 
    deployment = YAML.load_file("#{current_dir}/deployment.yml.dist")
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  deployment["nodes"].each do |node|

    config.vm.box_check_update = false

    config.vm.define node["name"] do |seedvm|
      seedvm.vm.box = node["box"]
      seedvm.vm.hostname = node["name"]

      # Configure libvirt provider
      seedvm.vm.provider "libvirt" do |libvirt|
        libvirt.memory = node["ram"]
      end

      seedvm.vm.provider :virtualbox do |vb|
        vb.name = node["name"]
        vb.memory = node["ram"]
        vb.gui = false
        vb.customize ["modifyvm", :id, "--memory", node["ram"]]
        vb.customize ["modifyvm", :id, "--vram", "16"]
        vb.customize ["modifyvm", :id, "--cpus", node["cpu"]]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--ioapic", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      end

      if node.has_key?("networks")
        puts "Configure additional networks"
        node["networks"].each do |network|
          seedvm.vm.network "private_network", ip: network["ip"], virtualbox__intnet: network["name"]
        end
      end

      if node.has_key?("forwards")
        puts "Configure port forwarding"
        node["forwards"].each do |forward|
          seedvm.vm.network "forwarded_port", id: forward["name"], host: forward["host"], guest: forward["guest"], auto_correct: true
        end
      end

      if node.has_key?("ssh_pub_keys")
        puts "Execute on node ssh_pub_keys"
        node["ssh_pub_keys"].each do |ssh_pub_key|
          seedvm.vm.provision "shell", inline: <<-SHELL
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          SHELL
        end
      end

      if node.has_key?("provisioner_shell")
        node["provisioner_shell"].each do |provision|
          puts "Execute on node #{provision}"
          # seedvm.vm.provision "shell", inline: #{provision}
        end
      end
    end # node
  end # config
end
