# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

# Reading config from config.yaml
cfg = YAML.load_file('config.yaml')

IP1 = cfg['ip1']
IMAGE_NAME = cfg['image_name']
OCF_FILE_PATH = cfg['ocf_file_path']
UPLOAD_METHOD = cfg['upload_method']
HOST_NUM = cfg['host_number']

# Method of executing command
def shell_script(filename, env=[], args=[])
  shell_script_crafted = "/bin/bash -c \"#{env.join ' '} #{filename} #{args.join ' '} 2>/dev/null\""
  shell_script_crafted
end

# Rabbitmq setup
rabbit_primitive_setup = shell_script("/vagrant/conf_rabbit_primitive.sh")
rabbit_ocf_setup = shell_script("/vagrant/conf_rabbit_ocf.sh")


Vagrant.configure(2) do |config|
  
  config.vm.box = IMAGE_NAME

  config.vm.define "node1", primary: true do |config|
    config.vm.host_name = "node1"
    corosync_setup = shell_script("/vagrant/conf_corosync.sh", [], ["#{IP1}.2"])
    config.vm.network :private_network, ip: "#{IP1}.2", :mode => 'nat'
    config.vm.provision "shell", run: "always", inline: corosync_setup, privileged: true
    config.vm.provision "shell", run: "always", inline: rabbit_ocf_setup, privileged: true
    config.vm.provision "shell", run: "always", inline: rabbit_primitive_setup, privileged: true
    
  end

  HOST_NUM.times do |i|
    host_index = i + 2
    ip = i + 3
    if ip < 254
      config.vm.define "node#{host_index}" do |config|
         config.vm.host_name = "node#{host_index}"
         # wait 2 seconds for the first corosync node
         corosync_setup = shell_script("/vagrant/conf_corosync.sh", [], ["#{IP1}.#{ip}", 2])
         config.vm.network :private_network, ip: "#{IP1}.#{ip}", :mode => 'nat'
         config.vm.provision "shell", run: "always", inline: corosync_setup, privileged: true
         config.vm.provision "shell", run: "always", inline: rabbit_ocf_setup, privileged: true
      end
    end
  end

end
