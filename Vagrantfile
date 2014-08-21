# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.require_version ">= 1.5.0"

# Size of the CoreOS cluster created by Vagrant
$num_instances=3

# Official CoreOS channel from which updates should be downloaded
$update_channel='stable'

# Setting for VirtualBox VMs
$vb_memory = 1024
$vb_cpus = 1

BASE_IP_ADDR  = ENV['BASE_IP_ADDR'] || "192.168.65"
ETCD_DISCVERY = "#{BASE_IP_ADDR}.101"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "coreos-%s" % $update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % $update_channel

  config.vm.define "discovery" do |discovery|
    discovery.vm.hostname = "discovery"

    discovery.vm.network :private_network, ip: ETCD_DISCVERY

    discovery.vm.provision :file, source: "./discovery", destination: "/tmp/vagrantfile-user-data"

    discovery.vm.provision :shell do |sh|
      sh.privileged = true
      sh.inline = <<-EOT
        mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/
      EOT
    end
  end

  (1..$num_instances).each do |i|
    config.vm.define "core-#{i}" do |core|
      config.vm.provider :virtualbox do |vb|
        vb.memory = $vb_memory
        vb.cpus = $vb_cpus
      end

      core.vm.hostname = "core-#{i}"

      core.vm.network :forwarded_port, guest: 4001, host: "400#{i}".to_i
      core.vm.network :forwarded_port, guest: 8080, host: "818#{i}".to_i

      core.vm.network :private_network, ip: "#{BASE_IP_ADDR}.#{i+1}"

      core.vm.provision :file, source: "./user-data", destination: "/tmp/vagrantfile-user-data"

      core.vm.provision :shell do |sh|
        sh.privileged = true
        sh.inline = <<-EOT
          sed -e "s/%NAME%/core-#{i}/g" -i /tmp/vagrantfile-user-data
          sed -e "s/%ETCD_DISCVERY%/#{ETCD_DISCVERY}/g" -i /tmp/vagrantfile-user-data
          mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/
        EOT
      end
    end
  end
end
