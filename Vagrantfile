# -*- mode: ruby -*-
# vi: set ft=ruby :

##
## variables
##
## Note: multiple vagrant plugins follow the following syntax:
##
##       required_plugins = %w(plugin1 plugin2 plugin3)
##
required_plugins           = %w(vagrant-vbguest)
plugin_installed           = false
project_root               = '/vagrant'
agent_version              = 'v1.2.11'
server_version             = 'v2.1.5'
docker_version_c           = '18.09.0'
docker_version_u           = '18.06.1*'
server_ip                  = '192.168.0.30'
agent_ip                   = '192.168.0.31'
server_port                = '7890'
server_https_port          = '7895'
server_internal_port       = '8890'
server_internal_https_port = '8895'

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
    :ram => 4096,
    :cpu => 4
  },
  {
    :hostname => 'rancher-agent',
    :ip => agent_ip,
    :box => 'ubuntu/bionic64',
    :ram => 2048,
    :cpu => 2
  }
]

## create vbox machines
Vagrant.configure(2) do |config|
    servers.each do |machine|
        ## shared files + folders
        config.vm.define machine[:hostname] do |node|
            ## virtualbox configurations
            node.vm.box = machine[:box]
            node.vm.hostname = machine[:hostname]
            node.vm.network 'private_network', ip: machine[:ip]
            node.vm.provider 'virtualbox' do |vb|
                vb.customize ['modifyvm', :id, '--memory', machine[:ram]]
            end

            ## pre-docker dependencies
            if machine[:hostname] == 'rancher-server'
                node.vm.network :forwarded_port, guest: server_internal_port, host: server_port
                node.vm.network :forwarded_port, guest: server_internal_https_port, host: server_https_port
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
                mkdir -p #{project_root}/logs
                dos2unix #{project_root}/utility/*
                chmod u+x #{project_root}/utility/*
                cd #{project_root}/utility
                ./install-rancher-cli >> #{project_root}/logs/install-rancher-cli.txt 2>&1
            SHELL

            ## install rancher server + agent
            if machine[:hostname] == 'rancher-server'
                node.vm.provision 'shell', inline: <<-SHELL
                    sudo yum -y update
                    cd #{project_root}/utility
                    ./install-docker #{docker_version_c} >> #{project_root}/logs/install-docker-centos7.txt 2>&1
                    ./install-rancher-server #{server_version} #{server_internal_port} #{server_internal_https_port} >> #{project_root}/logs/install-rancher-server.txt 2>&1
                    systemctl enable firewalld
                    systemctl start firewalld
                    firewall-cmd --zone=public --permanent --add-port=#{server_internal_port}/tcp
                    firewall-cmd --zone=public --permanent --add-port=#{server_internal_https_port}/tcp
                    firewall-cmd --reload
                SHELL

            else
                node.vm.provision 'shell', inline: <<-SHELL
                    sudo apt-get -y update
                    cd #{project_root}/utility
                    ./install-docker #{docker_version_u} >> #{project_root}/logs/install-docker-xenial.txt 2>&1
                    ./install-rancher-agent #{agent_version} #{server_ip} #{server_internal_port} #{server_internal_https_port} >> #{project_root}/logs/install-rancher-agent.txt 2>&1
                SHELL
            end
        end
    end
end
