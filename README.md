# Обновление ядра Linux в дистрибутиве Ubuntu # 

### Обновить ядро Linux в дистрибутиве Ubuntu из исходных кодов ядра. #####
&emsp; Для выполнения задания использовался образ (бокс) ubuntu/jammy64 версии 20240301.0.0,<br/> 
полученный по ссылке: <https://app.vagrantup.com/ubuntu/boxes/jammy64/><br/> 
&emsp; Для запуска виртуальной машины исполнен Vagrantfile следующего содержания:

``` ruby

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

  # обновление дерева пакетов
  sudo apt update

  # установка необходимых пакетов для компиляции исходных кодов ядра
  sudo apt install -y libncurses5-dev dwarves build-essential bison flex libssl-dev libelf-dev

  # исправление нарушенных зависимостей, возникших при установке пакетов
  sudo apt -f install

  # переход в общую между хостовой и виртуальной машинами директорию, которая будет использоваться, как рабочая
  cd /vagrant

  # получение архива с исходными кодами ядра
  wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.5.tar.gz

  # распаковка архива с исходными кодами ядра
  tar -xzf linux-6.5.tar.gz

  # переход в директорию с исходными кодами ядра
  cd /vagrant/linux-6.5

  # копирование конфигурационного файла установленного/старого ядра 
  cp /boot/config-5.15.0-97-generic .config

  # генерирование конфигурационного файла на основе текущей конфигурации ядра и загруженных модулей, 
  # отключается любая опция модуля, которая не нужна для загруженных модулей
  make localmodconfig

  # Устранение возникающей при компиляции ошибки "FAILED: load BTF from vmlinux: No such file or directory" 
  scripts/config --set-val CONFIG_DEBUG_INFO_BTF n

  # Блок команд отключающих в конфигурации ядра заверение сертификатами скомпилированных модулей ядра 
  scripts/config --disable SYSTEM_TRUSTED_KEYS
  scripts/config --disable SYSTEM_REVOCATION_KEYS

  # компилирование исходных кодов ядра
  make -j4

  # установка модулей ядра
  sudo make modules_install

  # установка ядра
  sudo make install

  # перезагрузка виртуального хоста
  sudo reboot
  SHELL
end

```
Вывод команды до обновления ядра:
```shell
vagrant@kernel-update:~$ uname -r
5.15.0-97-generic
```	
Вывод команды после обновления ядра:
```shell
vagrant@kernel-update:~$ uname -r 
6.5.0
```
&emsp; Для бокса ubuntu/jammy64 версии 20240301.0.0 после установки нового ядра манипуляции по восстановлению VirtualBox Shared Folders не потребовались. Указанный ресурс сохранил свою доступность и работоспособность.
