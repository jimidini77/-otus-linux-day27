# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
    :box_name => "centos/7",
    :vm_name => "inetRouter",
    :net => [
      {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"}, # /30
      {ip: '192.168.56.101', adapter: 3}
    ]
  },

  :centralRouter => {
    :box_name => "centos/7",
    :vm_name => "centralRouter",
    :net => [
      {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},# /30
      {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},   # /28
      {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},    # /28
      {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},    # /26
      {ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-router-net"},    # /28
      {ip: '192.168.255.5', adapter: 7, netmask: "255.255.255.252", virtualbox__intnet: "office2-router-net"},    # /26
      {ip: '192.168.56.102', adapter: 8}
    ]
  },
  
  :centralServer => {
    :box_name => "centos/7",
    :net => [
      {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},     # /28
      {ip: '192.168.56.103', adapter: 3}
    ]
  },

  :Office1Router => {
    :box_name => "centos/7",
    :vm_name => "Office1Router",
    :net => [
      {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-router-net"},# /30
      {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev1-net"},   # /26
      {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test1-net"},    # /26
      {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "manager1-net"},    # /26
      {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "office1-net"},    # /26
      {ip: '192.168.56.104', adapter: 7}
    ]
  },

:Office1Server => {
    :box_name => "centos/7",
    :vm_name => "Office1Server",    
    :net => [
      {ip: '192.168.2.130', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "manager1-net"},     # /28
      {ip: '192.168.56.105', adapter: 3}
    ]
},

:Office2Router => {
  :box_name => "centos/7",
  :vm_name => "Office2Router",
  :net => [
    {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2-router-net"},# /30
    {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},   # /26
    {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test2-net"},    # /26
    {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office2-net"},    # /26
    {ip: '192.168.56.106', adapter: 6}
  ]
},

:Office2Server => {
  :box_name => "centos/7",
  :vm_name => "Office2Server",    
  :net => [
    {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},     # /28
    {ip: '192.168.56.107', adapter: 3}
  ]
},

}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s

      boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", ipconf
      end
      
      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL

      if boxconfig[:vm_name] == "Office2Server"
        box.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/provision.yml"
          ansible.inventory_path = "ansible/hosts"
          ansible.host_key_checking = "false"
          ansible.limit = "all"
        end
      end

    end

  end
  
  
end
