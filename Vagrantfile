# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
#
#svc_net_prefix = '192.168.202'

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  #
  #
  #config.vm.box = "rhcos/4.4.3"

  # Mac addresses calculated using
  # gen-virt-mac-from-ip () {
  #   printf '52:54:00:%02X:%02X:%02X\n' $(echo $1| cut -d'.' --output-delimiter=' ' -f2,3,4)
  # }

  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.provider :vmware_fusion do |v|
    v.linked_clone = false
    v.gui = true
    v.vmx["numvcpus"] = "4"
    v.vmx["virtualHW.version"] = "19"
    v.vmx["ethernet0.present"] = "TRUE"
    v.vmx["ethernet0.vnet"] = "vmnet7"
    v.vmx["ethernet0.connectiontype"] = "custom"
    v.vmx["ethernet0.opromsize"] = "262144"
    v.vmx["ethernet0.virtualDev"] = "e1000e"
    v.vmx["e1000ebios.filename"] = File.dirname(__FILE__)+"/808610d3.mrom"
    v.vmx["tools.synctime"] = "TRUE"
  end


  config.vm.define :lb, autostart: false do |node|
    node.vm.box = "centos/7"
    node.vm.network :private_network,
      :ip => "192.168.100.2"
    node.vm.provider :vmware_fusion do |v|
      v.vmx["displayname"] = "okd-lb"
      v.vmx["memsize"] = "2048"
      v.vmx["numvcpus"] = "2"
      v.vmx["ethernet0.present"] = "TRUE"
      v.vmx["ethernet0.connectiontype"] = "nat"
      v.vmx["ethernet1.present"] = "TRUE"
      v.vmx["ethernet1.vnet"] = "vmnet7"
      v.vmx["ethernet1.connectiontype"] = "custom"
    end
    node.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/lb.yml"
    end
  end


  config.vm.define :bootstrap, autostart: false do |node|
    node.vm.disk :disk, size: "100GB", primary: true
    node.vm.box = "centos/7"    # This is solely because Vagrant <-> VMWare Fusion requires a box be set even though iPXE takes over
    # Note that the (automatically created) hard drive for this box needs to be SATA/SCSI -- not NVME
    node.ssh.private_key_path = File.dirname(__FILE__)+"/ssh_key/id_ed25519"
    node.ssh.username = "core"
    node.vm.provider :vmware_fusion do |v|
      v.vmx["displayname"] = "okd-bootstrap"
      v.vmx["memsize"] = "16384"
      v.vmx["ethernet0.addressType"] = "static"
      v.vmx["ethernet0.address"] = "52:54:00:A8:64:05"  # 192.168.100.5
      v.vmx["bios.bootOrder"] = "ethernet0"
    end
  end

  mac = [
    '52:54:00:A8:64:0A', # 192.168.100.10
    '52:54:00:A8:64:0B', # 192.168.100.11
    '52:54:00:A8:64:0C', # 192.168.100.12
  ]

  (0..2).each do |node_num|
    config.vm.define "cp#{node_num}", autostart: false do |node|
      node.vm.disk :disk, size: "100GB", primary: true
      node.vm.box = "centos/7"    # This is solely because Vagrant <-> VMWare Fusion requires a box be set even though iPXE takes over
      # Note that the (automatically created) hard drive for this box needs to be SATA/SCSI -- not NVME
      node.ssh.private_key_path = File.dirname(__FILE__)+"/ssh_key/id_ed25519"
      node.ssh.username = "core"
      node.vm.provider :vmware_fusion do |v|
        v.vmx["displayname"] = "okd-cp#{node_num}"
        v.vmx["memsize"] = "16384"
        v.vmx["ethernet0.addressType"] = "static"
        v.vmx["ethernet0.address"] = mac[node_num]  # 192.168.100.5
        v.vmx["bios.bootOrder"] = "ethernet0"
      end
    end
  end

  (0..2).each do |node_num|
    config.vm.define "worker#{node_num}", autostart: false do |node|
      node.vm.box = "centos/7"    # This is solely because Vagrant <-> VMWare Fusion requires a box be set even though iPXE takes over
      # Note that the (automatically created) hard drive for this box needs to be SATA/SCSI -- not NVME
      node.ssh.private_key_path = File.dirname(__FILE__)+"/ssh_key/id_ed25519"
      node.ssh.username = "core"
      node.vm.provider :vmware_fusion do |v|
        v.vmx["displayname"] = "okd-worker#{node_num}"
        v.vmx["memsize"] = "8192"
        v.vmx["bios.bootOrder"] = "ethernet0"
      end
    end
  end
end
