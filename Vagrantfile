# -*- mode: ruby -*-
# vi: set ft=ruby :

##
## variables
##
## Note: multiple vagrant plugins follow the following syntax:
##
##       required_plugins = %w(plugin1 plugin2 plugin3)
##
required_plugins  = %w(vagrant-vbguest)
plugin_installed  = false
project_root      = '/vagrant'

## install vagrant plugins
required_plugins.each do |plugin|
  unless Vagrant.has_plugin? plugin
    system "vagrant plugin install #{plugin}"
    plugin_installed = true
  end
end

## restart Vagrant: if new plugin installed
if plugin_installed == true
  exec "vagrant #{ARGV.join(' ')}"
end

## configurations
servers=[
  {
    :hostname => 'rancher-server',
    :ip => '192.168.0.30',
    :box => 'centos/7',
    :ram => 3072,
    :cpu => 4
  },
  {
    :hostname => 'rancher-agent',
    :ip => '192.168.0.31',
    :box => 'ubuntu/bionic64',
    :ram => 2048,
    :cpu => 4
  }
]

## create vbox machines
Vagrant.configure(2) do |config|
    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            node.vm.box = machine[:box]
            node.vm.hostname = machine[:hostname]
            node.vm.network 'private_network', ip: machine[:ip]
            node.vm.provider 'virtualbox' do |vb|
                vb.customize ['modifyvm', :id, '--memory', machine[:ram]]
            end

            if machine[:hostname] = 'rancher-server'
                node.vm.provision 'shell', inline: <<-SHELL
                    sudo yum install -y dos2unix
                SHELL
            else
                node.vm.provision 'shell', inline: <<-SHELL
                    sudo apt-get install -y dos2unix
                SHELL
            end

            node.vm.provision 'shell', inline: <<-SHELL
                dos2unix "#{project_root}"/utility/*
                chmod u+x "#{project_root}"/utility/*
                cd "#{project_root}"/utility
                ./install-docker '18.03.0'
            SHELL
        end
    end
end
