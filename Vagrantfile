# -*- mode: ruby -*-
# vim: set ft=ruby :

# Добавление директории HOME
home = ENV['HOME']

# Описание виртуальной машины
MACHINES = {
  :otuslinux => {
    :box_name => "generic/ubuntu2204",
    :net => [
      ["192.168.56.101", 2, "255.255.255.0"],
    ],
	  :disks => {
		  :sata1 => {
			  :dfile => './sata5.vdi',
			  :size => 10240,
			  :port => 1
		  },
		  :sata2 => {
        :dfile => './sata6.vdi',
        :size => 2048, # Megabytes
			  :port => 2
		  },
      :sata3 => {
        :dfile => './sata7.vdi',
        :size => 1024,
        :port => 3
      },
      :sata4 => {
        :dfile => './sata8.vdi',
        :size => 1024, # Megabytes
        :port => 4
      },
      :sata5 => {
        :dfile => './sata9.vdi',
        :size => 250, # Megabytes
        :port => 5
      }
	  }
  }
}


# Конфигурация виртуальной машины
Vagrant.configure("2") do |config|
# Перебирает каждый элемент хеша MACHINES. boxname — это ключ (в нашем случае :test_mdadm), а boxconfig — значение (конфигурация машины).
  MACHINES.each do |boxname, boxconfig|
# Определение конкретной машины
# Определяет конкретную машину с именем boxname (в нашем случае :test_mdadm). Все последующие операции будут относиться к этой машине.
    config.vm.define boxname do |box|
# Устанавливает имя бокса и хостнейм для виртуальной машины. boxconfig[:box_name] берет значение из конфигурации машины (в данном случае "generic/ubuntu2204").
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
# Настраивает частную сеть с указанным IP-адресом. Если в конфигурации есть параметр :public, то также настраивается публичная сеть.
      boxconfig[:net].each do |ipconf|
        box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2])
      end
#      box.vm.network "private_network", ip: boxconfig[:ip_addr]
#      if boxconfig.key?(:public)
#        box.vm.network "public_network", boxconfig[:public]
#      end
# Указывает, что далее идут настройки для провайдера VirtualBox. Устанавливает размер оперативной памяти в 1024 МБ.
        box.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "1024"]
# Создание и подключение дисков
# Проходит по каждому диску в конфигурации машины. Если файл диска еще не существует, создает его с указанными параметрами. После этого устанавливает переменную needsController в true, чтобы позже добавить к контроллеру SATA.
        needsController = false
        boxconfig[:disks].each do |dname, dconf|
          unless File.exist?(dconf[:dfile])
	    vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
            needsController = true
          end
        end
# Если хотя бы один диск был создан, добавляет контроллер SATA и подключает все созданные диски к этому контроллеру.    
        if needsController == true
# ВАЖНО. Строка создаёт SATA Controller. Т.к. мы используем готовый BOX generic/ubuntu2204, то там SATA Controller уже создан и при выполнении выдаёт ошибку. Дальнейшее добавление дисков не происходит. В связи с чем создавать контроллер НЕ НУЖНО.
      #vb.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata" ]
# Подключает все созданные диски к этому контроллеру.                     
          boxconfig[:disks].each do |dname, dconf|
            vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
          end
        end
      end
# Запуск playbook Ansible
#      config.vm.provision "ansible" do |ansible|
#        ansible.playbook = "playbook.yml"
#      end
# Выполнение команды SHELL
      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        apt install -y mdadm smartmontools hdparm gdisk
      SHELL
    end
  end
end