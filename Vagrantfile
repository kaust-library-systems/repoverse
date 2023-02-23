# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  config.vm.define "repoverse" do |repoverse|
    repoverse.vm.hostname = "repoverse"
    repoverse.vm.provider :virtualbox do |vb|
      vb.memory = 8192
      vb.cpus = 4
    end
  end
end
