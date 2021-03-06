# -*- mode: ruby -*-
# vim: set ft=ruby ts=2 sw=2 et sts=2 :

VAGRANTFILE_API_VERSION = "2"
CURRENT_USER = `whoami`.chomp

# our script to install the stackstrap states on the vagrant box
$stackstrap_salt_script = <<SCRIPT
# install git
salt-call pkg.install git

# setup our /srv directory
# TODO: make the GIT URL & ref configurable
cd /tmp
git clone https://github.com/stackstrap/stackstrap-salt.git stackstrap-salt
cd stackstrap-salt
git checkout v2014.2.6
git archive v2014.2.6 --prefix=/srv/ | (cd /; tar xf -)
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.hostname = "{{ name }}-#{CURRENT_USER}"

  config.vm.box = "stackstrap/ubuntu-trusty"

  config.vm.network :public_network

  config.ssh.forward_agent = true

  config.vm.synced_folder ".", "/home/vagrant/domains/{{ name }}",
    owner: "vagrant",
    group: "vagrant",
    mount_options: ["dmode=755,fmode=644"]

  # provision our box with salt but do not run the highstate yet
  # we still need to setup our stackstrap repository after salt is installed
  config.vm.provision :salt do |salt|
    salt.minion_config = "salt/minion"
    salt.run_highstate = false
    salt.install_type = "git"
    salt.install_args = "v0.17.5"
  end

  # get our stackstrap salt repository setup
  config.vm.provision :shell, inline: $stackstrap_salt_script

  # now run the highstate
  config.vm.provision :shell, inline: "salt-call state.highstate"

  config.vm.provision :shell,
    inline: "echo; echo; echo \"Your development environment is now configured\"; echo \"You can access it at http://{{ name }}-#{CURRENT_USER}.local/\"; echo"
end
