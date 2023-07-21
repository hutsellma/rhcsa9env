VAGRANTFILE_API_VERSION = "2"
VAGRANT_DISABLE_VBOXSYMLINKCREATE = "1"
file_to_disk1 = './disk-0-1.vdi'
file_to_disk2 = './disk-0-2.vdi'
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Use same SSH key for each machine
  config.ssh.insert_key = false
  config.vm.box_check_update = false
  config.vm.boot_timeout = 900
  
  # Repo Configuration
  config.vm.define "repo" do |repo|
    repo.vm.box = "micandhut/rhel9repo"
    repo.vm.provision :shell, :inline => "sudo rm -f /EMPTY;"
    repo.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;"
    repo.vm.provision :shell, :inline => "yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -y; sudo yum install -y sshpass python3-pip python3-devel httpd sshpass vsftpd createrepo"
    repo.vm.provision :shell, :inline => " python3 -m pip install -U pip --no-warn-script-location ; python3 -m pip install pexpect --no-warn-script-location; python3 -m pip install ansible --no-warn-script-location"
    repo.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/", rsync__exclude: "*.vdi"
    repo.vm.network "private_network", ip: "192.168.55.149"
  
    repo.vm.provider "virtualbox" do |repo|
      repo.memory = "1024"
    end
    repo.vm.provision :shell, :inline => "nmcli con mod 'System enp0s8' ipv4.addresses 192.168.55.149/24 ipv4.gateway 192.168.55.1 ipv4.dns 8.8.8.8 ipv4.method manual"

    repo.vm.provision :ansible_local do |ansible|
      ansible.playbook = "/vagrant/playbooks/repo.yml"
      ansible.install = false
      ansible.compatibility_mode = "2.0"
      ansible.inventory_path = "/vagrant/inventory"
      ansible.config_file = "/vagrant/ansible.cfg"
      ansible.limit = "all"
    end
  end

  # Server 1 Configuration
  config.vm.define "server1" do |server1|
    server1.vm.box = "micandhut/rhel9node"
    server1.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/", rsync__exclude: "*.vdi"
    server1.vm.network "private_network", ip: "192.168.55.150"
    
    server1.vm.provider :virtualbox do |server1|
      server1.customize ['modifyvm', :id,'--memory', '2048']
    end

      server1.vm.provision :ansible_local do |ansible|
      ansible.playbook = "/vagrant/playbooks/server1.yml"
      ansible.install = false
      ansible.compatibility_mode = "2.0"
      ansible.inventory_path = "/vagrant/inventory"
      ansible.config_file = "/vagrant/ansible.cfg"
      ansible.limit = "all"
    end
    server1.vm.provision :shell, :inline => "reboot", run: "always"
  end

    # Server 2 Configuration
    config.vm.define "server2", run: "always" do |server2|
      server2.vm.box = "micandhut/rhel9node"
      server2.vm.network "private_network", ip: "192.168.55.151"
      server2.vm.network "private_network", ip: "192.168.55.175"
      server2.vm.network "private_network", ip: "192.168.55.176"
      server2.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/", rsync__exclude: "*.vdi"
      server2.vm.provider "virtualbox" do |server2|
        server2.memory = "2404"
  
        unless File.exist?(file_to_disk1)
          server2.customize ['createhd', '--filename', file_to_disk1, '--variant', 'Fixed', '--size', 8 * 1024]
        end
    
        unless File.exist?(file_to_disk2)
          server2.customize ['createhd', '--filename', file_to_disk2, '--variant', 'Fixed', '--size', 8 * 1024]
        end
  
        server2.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 3, '--device', 0, '--type', 'hdd', '--medium', file_to_disk1]
        server2.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 4, '--device', 0, '--type', 'hdd', '--medium', file_to_disk2]
      end
    
      server2.vm.provision :ansible_local do |ansible|
        ansible.playbook = "/vagrant/playbooks/server2.yml"
        ansible.install = false
        ansible.compatibility_mode = "2.0"
        ansible.inventory_path = "/vagrant/inventory"
        ansible.config_file = "/vagrant/ansible.cfg"
        ansible.limit = "all"
      end
      server2.vm.provision :shell, :inline => "reboot", run: "always"
    end

end
