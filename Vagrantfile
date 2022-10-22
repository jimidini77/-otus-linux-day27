# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
    :box_name => "centos/7",
    :vm_name => "inetRouter",
    :net => [
      {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"} # /30
    ]
  },

  :centralRouter => {
    :box_name => "centos/7",
    :vm_name => "centralRouter",
    :net => [
      {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},# /30
      {ip: '192.168.0.1',   adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},   # /28
      {ip: '192.168.0.33',  adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},    # /28
      {ip: '192.168.0.65',  adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},    # /26
      {ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-router-net"},    # /28
      {ip: '192.168.255.5', adapter: 7, netmask: "255.255.255.252", virtualbox__intnet: "office2-router-net"}    # /26
    ]
  },
  
  :centralServer => {
    :box_name => "centos/7",
    :net => [
      {ip: '192.168.0.2',  adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"}     # /28
    ]
  },

  :Office1Router => {
    :box_name => "centos/7",
    :vm_name => "Office1Router",
    :net => [
      {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-router-net"},# /30
      {ip: '192.168.2.1',    adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev1-net"},   # /26
      {ip: '192.168.2.65',   adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test1-net"},    # /26
      {ip: '192.168.2.129',  adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "manager1-net"},    # /26
      {ip: '192.168.2.193',  adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "office1-net"}    # /26
    ]
  },

:Office1Server => {
    :box_name => "centos/7",
    :vm_name => "Office1Server",    
    :net => [
      {ip: '192.168.2.130',  adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "manager1-net"}     # /28
    ]
},

:Office2Router => {
  :box_name => "centos/7",
  :vm_name => "Office2Router",
  :net => [
    {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2-router-net"},# /30
    {ip: '192.168.1.1',    adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},   # /26
    {ip: '192.168.1.129',   adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test2-net"},    # /26
    {ip: '192.168.1.193',  adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office2-net"}    # /26
  ]
},

:Office2Server => {
  :box_name => "centos/7",
  :vm_name => "Office2Server",    
  :net => [
    {ip: '192.168.1.2',  adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"}     # /28
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
      
      case boxname.to_s
      when "inetRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
          sysctl -p
          yum install -y iptables iptables-services
          systemctl enable iptables.service
          systemctl start iptables.service
          iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
          iptables -D INPUT -j REJECT --reject-with icmp-host-prohibited
          iptables -D FORWARD -j REJECT --reject-with icmp-host-prohibited
          iptables-save > /etc/sysconfig/iptables
          systemctl restart iptables.service
          echo "192.168.255.4/30 via 192.168.255.2" > /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.255.8/30 via 192.168.255.2" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.0.0/24 via 192.168.255.2" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.1.0/24 via 192.168.255.2" >> /etc/sysconfig/network-scripts/route-eth1
          echo "192.168.2.0/24 via 192.168.255.2" >> /etc/sysconfig/network-scripts/route-eth1
          reboot
          SHELL
      when "centralRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
          sysctl -p
          yum install -y iptables iptables-services
          systemctl enable iptables.service
          systemctl start iptables.service
          iptables -D FORWARD -j REJECT --reject-with icmp-host-prohibited
          iptables-save > /etc/sysconfig/iptables
          systemctl restart iptables.service
          echo "192.168.2.0/24 via 192.168.255.10" > /etc/sysconfig/network-scripts/route-eth5
          echo "192.168.1.0/24 via 192.168.255.6" > /etc/sysconfig/network-scripts/route-eth6
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          reboot
          SHELL
      when "centralServer"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          reboot
          SHELL
      when "Office1Router"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
          sysctl -p
          yum install -y iptables iptables-services
          systemctl enable iptables.service
          systemctl start iptables.service
          iptables -D FORWARD -j REJECT --reject-with icmp-host-prohibited
          iptables-save > /etc/sysconfig/iptables
          systemctl restart iptables.service
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.255.9" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          reboot
          SHELL
      when "Office1Server"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          reboot
          SHELL
      when "Office2Router"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
          sysctl -p
          yum install -y iptables iptables-services
          systemctl enable iptables.service
          systemctl start iptables.service
          iptables -D FORWARD -j REJECT --reject-with icmp-host-prohibited
          iptables-save > /etc/sysconfig/iptables
          systemctl restart iptables.service
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.255.5" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          reboot
          SHELL
      when "Office2Server"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          reboot
          SHELL
      end

    end

  end
  
  
end
