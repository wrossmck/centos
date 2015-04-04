# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.vm.define "test" do |c|
    c.vm.box = "centos.box"
    config.ssh.insert_key = true
  end
  config.vm.define "cenots" do |c|
    c.vm.box = "chef/centos-7.0"
    c.vbguest.no_install = true

    c.vm.provision "shell", inline: <<-SHELL
      sudo yum upgrade -y
      sudo yum update -y
      sudo yum install -y nano docker \
      https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.2_x86_64.rpm \
      sudo yum clean all

      curl -L https://www.chef.io/chef/install.sh | sudo bash

      sudo groupadd docker
      sudo usermod -G docker vagrant
      sudo systemctl enable docker.service
      sudo systemctl enable docker
      sudo service docker start

      cd /vagrant/nanorc-master
      sudo make install-global
      sudo cp /vagrant/.nanorc /home/vagrant/.nanorc
      sudo cp /vagrant/.nanorc /root/.nanorc
    SHELL
  end

# NOTE: a vagrant reload must be done
# and the last part of the provisioning must also happen
# in order to load the vbox guest additions correctly
# that is why it happens in this "second vm step"
  config.vm.define "package" do |c|
    c.vm.box = "package.box"
    c.ssh.insert_key = false

    # c.vm.provider "virtualbox" do |vb|
    #   vb.memory = "4024"
    #   vb.cpus = "4"
    # end

    # Massive cleanup effort pilfered from Bento
    c.vm.provision "shell", inline: <<-SHELL

      curl -L "https://raw.githubusercontent.com/chef/bento/master/packer/scripts/centos/cleanup.sh" | sudo bash
      curl -L "https://raw.githubusercontent.com/chef/bento/master/packer/scripts/common/minimize.sh" | sudo bash
      curl -L "https://raw.githubusercontent.com/chef/bento/master/packer/scripts/common/sshd.sh" | sudo bash
      curl -L "https://raw.githubusercontent.com/chef/bento/master/packer/scripts/common/sudoers.sh" | sudo bash
      curl -L "https://raw.githubusercontent.com/chef/bento/master/packer/scripts/common/vagrant.sh" | sudo bash

      # modified version of bento/packer/scripts/centos/fix-slow-dns.sh

      #!/bin/sh -eux
      ## https://access.redhat.com/site/solutions/58625 (subscription required)
      # add 'single-request-reopen' so it is included when /etc/resolv.conf is generated
      echo 'RES_OPTIONS="single-request-reopen"' >> /etc/sysconfig/network
      service network restart
      echo 'Slow DNS fix applied (single-request-reopen)'

      cat /dev/null > ~/.bash_history && history -c && history -w
    SHELL
  end
end
