# -*- mode: ruby -*-
# vi: set ft=ruby :
VM_NAME="skf-k8s"
DOMAIN="pvt.arlab.pw"
MAC=18
SUBNET="192.168.5"
IP="#{SUBNET}.#{MAC}"
NETMASK="24" 
GW="#{SUBNET}.1"
DNS="#{SUBNET}.1"

DEFAULT_MEM=2048
DEFAULT_CPU=2
WORKER_MEM=4096
WORKER_CPU=4

KEY_pub="ssh-rsa XXX"
UPASS='passsword_hash'

Vagrant.configure("2") do |config|
  config.vm.synced_folder "_sync", "/vagrant", type: "rsync"
  config.vm.provision :shell, privileged: false, inline: $install_initial

    config.vm.define :master do |master|
      master.vm.box = 'generic/ubuntu2004'
      master.vm.hostname = "#{VM_NAME}-cp.#{DOMAIN}"
      master.vm.provider :vmware_esxi do |esxi|
        esxi.guest_name = "#{VM_NAME}-cp"
        esxi.guest_memsize = "#{DEFAULT_MEM}"
        esxi.guest_numvcpus = "#{DEFAULT_CPU}"
        esxi.esxi_virtual_network = ['External']
        esxi.guest_mac_address = ["00:50:56:05:00:#{MAC}"]

        esxi.guest_disk_type = 'thin'
        esxi.guest_virtualhw_version = '14'
        #esxi.guest_nic_type = 'e1000'
        #esxi.guest_boot_disk_size = 11
        #esxi.guest_username = 'vagrant'
        esxi.debug = 'true'
        esxi.esxi_hostname = 'NucESXi'
        esxi.esxi_username = 'root'
        esxi.esxi_password = 'env:NucESXiproot' #OK
      end
    end

  # %w{worker1 worker2}
  %w{worker1}.each_with_index do |name, i|
    config.vm.define name do |worker|
      worker.vm.box = 'generic/ubuntu2004'
      worker.vm.hostname = "#{VM_NAME}-#{name}.#{DOMAIN}"
      worker.vm.provider :vmware_esxi do |esxi|
        esxi.guest_name = "#{VM_NAME}-#{name}"
        esxi.guest_memsize = "#{WORKER_MEM}"
        esxi.guest_numvcpus = "#{WORKER_CPU}"
        esxi.esxi_virtual_network = ['External']
        esxi.guest_mac_address = ["00:50:56:05:00:#{MAC + 1 + i}"]

        esxi.guest_disk_type = 'thin'
        esxi.guest_virtualhw_version = '14'
        #esxi.guest_nic_type = 'e1000'
        #esxi.guest_boot_disk_size = 11
        #esxi.guest_username = 'vagrant'
        esxi.debug = 'true'
        esxi.esxi_hostname = 'NucESXi'
        esxi.esxi_username = 'root'
        esxi.esxi_password = 'env:NucESXiproot' #OK
      end
    end
  end

end

### privileged false
  $install_initial = <<-SCRIPT
  printf " \n-= Initial deployment =-\n"
  # add access
  echo #{KEY_pub} >> /home/vagrant/.ssh/authorized_keys    
  sudo usermod -p "#{UPASS}" vagrant
  # fix DNS
  sudo sed -i 's/4\.2\.2.*]/#{DNS}]/g' /etc/netplan/*
  sudo netplan apply
  sudo sed -i 's/4\.2\.2.*$/#{DNS}/g' /etc/systemd/resolved.conf
  sudo sed -i 's/^DNSSEC/#DNSSEC/g' /etc/systemd/resolved.conf
  sudo systemctl restart systemd-resolved
  # clean /etc/hosts
  ip=$(ip addr show | grep -o "inet #{SUBNET}\.[0-9]*" | cut -d ' ' -f2)
  hn=$(hostname -f | cut -d '.' -f1)
  sudo sed -i '/localdomain/d' /etc/hosts
  sudo sed -i '/127\.0\.2/d' /etc/hosts
  sudo sed -i '/#{SUBNET}/d' /etc/hosts
  echo "$ip    $hn.#{DOMAIN} $hn" | sudo tee -a /etc/hosts
  #  sudo apt-get update && sudo apt-get upgrade -y
  hostname -f ; hostname -i
  printf "\n-= Initial deployment completed =-\n \n" 
SCRIPT
### end of $install_initial
