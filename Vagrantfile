$deployment_init_INSTALL = <<-SCRIPT
    sudo apt update
SCRIPT

Vagrant.configure("2") do |vm_conf|
    
    #free5g
    vm_conf.vm.define "free5g" do |config|
        config.vm.provider "virtualbox" do |v|
            v.memory = 4096
        end
        config.vm.box = "ubuntu/bionic64"
        config.vm.network "private_network", ip:  "172.16.0.12"
        config.vm.provision "shell", inline: $deployment_init_INSTALL
    end
end
