# -*- mode: ruby -*-
# vi: set ft=ruby :

##
## variables
##
## Note: multiple vagrant plugins follow the following syntax:
##
##       required_plugins = %w(plugin1 plugin2 plugin3)
##
required_plugins = %w(vagrant-vbguest)
plugin_installed = false
project_root     = '/vagrant'
agent_version    = 'v1.2.10'
server_version   = 'v2.1.4'
docker_version   = '18.09.0'
server_ip        = '192.168.0.30'
agent_ip         = '192.168.0.31'

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
    :ip => server_ip,
    :box => 'centos/7',
    :ram => 3072,
    :cpu => 4
  },
  {
    :hostname => 'rancher-agent',
    :ip => agent_ip,
    :box => 'ubuntu/bionic64',
    :ram => 2048,
    :cpu => 4
  }
]

## create vbox machines
Vagrant.configure(2) do |config|
    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
            ## virtualbox configurations
            node.vm.box = machine[:box]
            node.vm.hostname = machine[:hostname]
            node.vm.network 'private_network', ip: machine[:ip]
            node.vm.provider 'virtualbox' do |vb|
                vb.customize ['modifyvm', :id, '--memory', machine[:ram]]
            end

            ## pre-docker dependencies
            if machine[:hostname] = 'rancher-server'
                node.vm.provision 'shell', inline: <<-SHELL
                    sudo yum install -y dos2unix
                SHELL
            else
                node.vm.provision 'shell', inline: <<-SHELL
                    sudo apt-get install -y dos2unix
                SHELL
            end

            ## utility executable: install docker + rancher-cli
            node.vm.provision 'shell', inline: <<-SHELL
                dos2unix "#{project_root}"/utility/*
                chmod u+x "#{project_root}"/utility/*
                cd "#{project_root}"/utility
                ./install-docker #{docker_version}
                ./install-rancher-cli
            SHELL

            ## install rancher server + agent
            if machine[:hostname] = 'rancher-server'
                node.vm.provision 'shell', inline: <<-SHELL
                    ./install-rancher-server #{server_version}
                SHELL

            else
                node.vm.provision 'shell', inline: <<-SHELL
                    ./install-rancher-agent #{agent_version}
                SHELL
            end
        end
    end
end
