# -*- mode: ruby -*-
# vi: set ft=ruby :

# This requires Vagrant 1.6.2 or newer (earlier versions can't reliably
# configure the Fedora 20 network stack).
Vagrant.require_version ">= 1.6.2"

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"


$bootstrap = <<SCRIPT
set -e

# Make it possible to install flocker-node
# XXX https://github.com/ClusterHQ/flocker/issues/306 for better secure downloads:
yum localinstall -y https://storage.googleapis.com/archive.clusterhq.com/fedora/clusterhq-release$(rpm -E %dist).noarch.rpm

# For reasons I do not understand this doesn't work (https://github.com/ClusterHQ/flocker/issues/510):
#yum install -y flocker-node
# So instead do it the stupid way:
yum install -y https://storage.googleapis.com/archive.clusterhq.com/fedora/20/x86_64/python-flocker-0.1.0-1.fc20.noarch.rpm https://storage.googleapis.com/archive.clusterhq.com/fedora/20/x86_64/flocker-node-0.1.0-1.fc20.noarch.rpm

# At the end of this bootstrap we'll (indirectly) ask Docker to write a very
# large file to its temporary directory.  /tmp is a small tmpfs mount which
# can't hold the file.  Convince Docker to write somewhere else instead.
echo "# Flocker-defined alternate temporary path to provide more temporary space." >> /etc/sysconfig/docker
echo "TMPDIR=/var/tmp" >> /etc/sysconfig/docker

systemctl enable docker
# Enabling Docker allows it to be started by socket activation, but it may
# already be running. Restart it to ensure that it picks up the new tmpdir
# configuration.
systemctl restart docker
systemctl enable geard
systemctl start geard

# Make it easy to authenticate as root
mkdir -p /root/.ssh
cp ~vagrant/.ssh/authorized_keys /root/.ssh

# Create a ZFS storage pool backed by a normal filesystem file.  This
# is a bad way to configure ZFS for production use but it is
# convenient for a demo in a VM.
mkdir -p /opt/flocker
truncate --size 1G /opt/flocker/pool-vdev
zpool create flocker /opt/flocker/pool-vdev

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "clusterhq/flocker-dev"
  config.vm.provision :shell, :inline => $bootstrap, :privileged => true

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  config.vm.define "node1" do |node1|
    node1.vm.network :private_network, :ip => "172.16.255.250"
    node1.vm.hostname = "node1"
  end

  config.vm.define "node2" do |node2|
    node2.vm.network :private_network, :ip => "172.16.255.251"
    node2.vm.hostname = "node2"
  end

end
