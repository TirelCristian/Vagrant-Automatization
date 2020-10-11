Vagrant.configure("2") do |config|

#vagrant plugin install vagrant-vbguest
#vagrant plugin install vagrant-hostsupdater
	
config.vm.box = "generic/rhel8"
	config.vm.define "NFS-Server" do |machine|
		machine.vm.synced_folder "PublicKey", "/PublicKey", :mount_options => ["ro"]
		machine.vm.hostname = "NFS-Server"
		machine.vm.network "private_network", ip: "192.168.33.50"
		machine.vm.provider "virtualbox" do |vb|
			vb.name = "NFS-Server"
			vb.gui = true
			vb.memory = "3072"
		end
		
		machine.vm.provision "shell", inline: <<-SHELL
				subscription-manager register --username cristian.tirel --password !!! --auto-attach
				sudo subscription-manager refresh
				sudo sed -i 's/AuthorizedKeysFile/#AuthorizedKeysFile/' /etc/ssh/sshd_config
				sudo cp /PublicKey/cheia.pub /home/vagrant/.ssh/authorized_keys2
				sudo chown vagrant:vagrant /home/vagrant/.ssh/ -R
				sudo chmod 600 /home/vagrant/.ssh/authorized_keys2
				sudo systemctl restart sshd
				cp /PublicKey/exemplu.pptx /home/vagrant/
		SHELL
	end

	(1..1).each do |exact|
		config.vm.define "Client#{exact}" do |machineK|
			machineK.vm.synced_folder "PublicKey", "/PublicKey", :mount_options => ["ro"]
			machineK.vm.hostname = "Client#{exact}"
			machineK.vm.network "private_network", ip: "192.168.33.20#{exact}"
			machineK.vm.provider "virtualbox" do |vb|
				vb.name = "Client#{exact}"
				vb.gui = true
				vb.memory = "3072"
			end
			
			machineK.vm.provision "shell", inline: <<-SHELL
				subscription-manager register --username cristian.tirel --password !!! --auto-attach
				sudo subscription-manager refresh
				sudo sed -i 's/AuthorizedKeysFile/#AuthorizedKeysFile/' /etc/ssh/sshd_config
				sudo cp /PublicKey/cheia.pub /home/vagrant/.ssh/authorized_keys2
				sudo chown vagrant:vagrant /home/vagrant/.ssh/ -R
				sudo chmod 600 /home/vagrant/.ssh/authorized_keys2
				sudo systemctl restart sshd
#				sudo yum -y groupinstall "Server with GUI"
#				sudo systemctl set-default graphical.target
			SHELL
		end
	end

	config.vm.define "Controller" do |machine|
 		machine.vm.synced_folder "Provisioning", "/Provisioning", :mount_options => ["ro"]
		machine.vm.hostname = "AnsibleController"
		machine.vm.network "private_network", ip: "192.168.33.100"
		machine.vm.provider "virtualbox" do |vb|
			vb.name = "Controller"
			vb.gui = true
			vb.memory = "3072"
		end

		machine.vm.provision "shell", inline: <<-SHELL
			subscription-manager register --username cristian.tirel --password !!! --auto-attach
			sudo subscription-manager refresh
			sudo mkdir /etc/ansible -p
			sudo cp /Provisioning/ansible.cfg /etc/ansible/ansible.cfg
			sudo chown root:root /etc/ansible/ansible.cfg
			sudo chmod 644 /etc/ansible/ansible.cfg
			sudo cp /Provisioning/cheia /home/vagrant/.ssh/cheia
			sudo chown vagrant:vagrant /home/vagrant/ -R
			sudo chmod 600 /home/vagrant/.ssh/cheia
		SHELL
		
		machine.vm.provision :ansible_local do |ansible|
			ansible.provisioning_path = "/Provisioning"
			ansible.playbook       = "playbook.yml"
			ansible.verbose        = true
			ansible.install        = true
			ansible.limit          = "all"
			ansible.inventory_path = "Inventory/Inventory.yml"
		end
	end
end