VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
      
  # config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", disabled: true
  
  config.vm.define :haproxy1 do |haproxy1|
    haproxy1.vm.box = "ubuntu/xenial64"
    haproxy1.vm.hostname = "haproxy1"
    haproxy1.vm.network :private_network, ip: "192.168.56.211"
    haproxy1.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", "2048"]
      v.customize ["modifyvm", :id, "--cpus", "2"]
      v.customize ["modifyvm", :id, "--ioapic", "on"]
      v.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
      v.gui = true
    end
    # config.vm.synced_folder "vagrant/chef-repo", "/home/ubuntu/chef-repo"
    # Install Chef-client inside Vbox guest VM
    haproxy1.vm.provision "shell", inline: "curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -v 13.2.20"
    # Use chef provisioning
    haproxy1.vm.provision "chef_zero" do |chef|
      # Specify the local paths where Chef data is stored
      chef.cookbooks_path = "cookbooks"
      chef.data_bags_path = "data_bags"
      chef.nodes_path = "cookbooks/nodes"
      #chef.roles_path = "roles"
  
      # Add a recipe
      chef.add_recipe "install_haproxy_keepalived"
  
      # Or maybe a role
      #chef.add_role "web"
    end
  end

  config.vm.define :haproxy2 do |haproxy2|
    haproxy2.vm.box = "ubuntu/xenial64"
    haproxy2.vm.hostname = "haproxy2"
    haproxy2.vm.network :private_network, ip: "192.168.56.212"
    haproxy2.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--memory", "2048"]
      v.customize ["modifyvm", :id, "--cpus", "2"]
      v.customize ["modifyvm", :id, "--ioapic", "on"]
      v.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
      v.gui = true
    end
    # config.vm.synced_folder "vagrant/chef-repo", "/home/ubuntu/chef-repo"
    # Install Chef-client inside Vbox guest VM
    haproxy2.vm.provision "shell", inline: "curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -v 13.2.20"
    # Use chef provisioning
    haproxy2.vm.provision "chef_zero" do |chef|
      # Specify the local paths where Chef data is stored
      chef.cookbooks_path = "cookbooks"
      chef.data_bags_path = "data_bags"
      chef.nodes_path = "cookbooks/nodes"
      #chef.roles_path = "roles"
  
      # Add a recipe
      chef.add_recipe "install_haproxy_keepalived"
  
      # Or maybe a role
      #chef.add_role "web"
    end
  end
end

