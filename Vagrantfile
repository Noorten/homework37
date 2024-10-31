# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        :vm_name => "inetRouter",
        :net => [
                   {type: "private_network", adapter: 2, auto_config: false, virtualbox__intnet: "router-net"},
                   {type: "private_network", adapter: 3, auto_config: false, virtualbox__intnet: "router-net"},
                   {type: "private_network", ip: '192.168.56.10', adapter: 8, netmask: '255.255.255.0'}

                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :vm_name => "centralRouter",
        :net => [
                   {type: "private_network", adapter: 2, auto_config: false, virtualbox__intnet: "router-net"},
                   {type: "private_network", adapter: 3, auto_config: false, virtualbox__intnet: "router-net"},
                   {type: "private_network", ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                   {type: "private_network", ip: '192.168.56.11', adapter: 8, netmask: '255.255.255.0'}
                ]
  },
  :office1Router => {
        :box_name => "centos/7",
        :vm_name => "office1Router",
        :net => [
                   {type: "private_network", ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                   {type: "private_network", adapter: 3, auto_config: false, virtualbox__intnet: "vlan1"},
                   {type: "private_network", adapter: 4, auto_config: false, virtualbox__intnet: "vlan1"},
                   {type: "private_network", adapter: 5, auto_config: false, virtualbox__intnet: "vlan2"},
                   {type: "private_network", adapter: 6, auto_config: false, virtualbox__intnet: "vlan2"},
                   {type: "private_network", ip: '192.168.56.20', adapter: 8, netmask: '255.255.255.0'}

                ]
  },
  :testClient1 => {
        :box_name => "centos/7",
        :vm_name => "testClient1",
        :net => [
                   {type: "private_network", adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                   {type: "private_network", ip: '192.168.56.21', adapter: 8, netmask: '255.255.255.0'}
                ]
  },
  :testServer1 => {
        :box_name => "centos/7",
        :vm_name => "testServer1",
        :net => [
                   {type: "private_network", adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                   {type: "private_network", ip: '192.168.56.22', adapter: 8, netmask: '255.255.255.0'}
                ]
  },
  :testClient2 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "testClient2",
        :net => [
                   {type: "private_network", adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                   {type: "private_network", ip: '192.168.56.31', adapter: 8, netmask: '255.255.255.0'}
                ]
  },
  :testServer2 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "testServer2",
        :net => [
                   {type: "private_network", adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                   {type: "private_network", ip: '192.168.56.32', adapter: 8, netmask: '255.255.255.0'}
                ]
  },

}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|
    
    config.vm.define boxname do |box|
   
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]

      config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
       end

      if boxconfig[:vm_name] == "testServer2"
       box.vm.provision "ansible" do |ansible|
        ansible.playbook = "templates/provision.yml"
        ansible.inventory_path = "templates/hosts"
        ansible.host_key_checking = "false"
        ansible.become = "true"
        ansible.limit = "all"
       end
      end

      boxconfig[:net].each do |ipconf|
	if ipconf[:ip]
	  box.vm.network ipconf[:type], ip: ipconf[:ip], adapter: ipconf[:adapter], netmask: ipconf[:netmask], virtualbox__intnet: ipconf[:virtualbox__intnet]
        else
          box.vm.network ipconf[:type], adapter: ipconf[:adapter], virtualbox__intnet: ipconf[:virtualbox__intnet], auto_config: ipconf[:auto_config]
        end
      end

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
	sed -i.bak 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
	sed -i.bak 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-* 
	sed -i 's/^#PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
	systemctl restart sshd.service
      SHELL
    end
  end
end
