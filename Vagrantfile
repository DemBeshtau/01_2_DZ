# -*- mode: ruby -*-
# vi: ft=ruby :

MACHINES = {
    :"kernel-update" => {
        :box_name => "ubuntu/jammy64",
        :box_version => "0",
        :cpus => 4,
        :memory => 4096,
    }
}

Vagrant.configure("2") do |config|
    MACHINES.each do |boxname, boxconfig|
        config.vm.define boxname do |box|
            box.vm.box = boxconfig[:box_name]
            box.vm.box_version = boxconfig[:box_version]
            box.vm.host_name = boxname.to_s;
            box.vm.provider "virtualbox" do |v|
	            v.memory = boxconfig[:memory]
                v.cpus = boxconfig[:cpus]
            end
        end    
    end
    config.vm.provision "shell", inline: <<-SHELL
    sudo apt update
    sudo apt install -y libncurses5-dev dwarves build-essential bison flex libssl-dev libelf-dev
    sudo apt -f install
    cd /vagrant
    wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.5.tar.gz
    tar -xzf linux-6.5.tar.gz
    cd /vagrant/linux-6.5
    cp /boot/config-5.15.0-97-generic .config
    make localmodconfig
    scripts/config --set-val CONFIG_DEBUG_INFO_BTF n
    scripts/config --disable SYSTEM_TRUSTED_KEYS
    scripts/config --disable SYSTEM_REVOCATION_KEYS
    make -j4
    sudo make modules_install
    sudo make install
    cd /vagrant 
    rm -r linux-6.5 linux-6.5.tar.gz
    sudo reboot
    SHELL
end
