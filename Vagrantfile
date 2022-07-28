$script_mysql = <<-SCRIPT
  apt-get update && \
  apt-get install -y mysql-server-5.7 && \
  mysql -e "create user 'phpuser'@'%' identified by 'pass';"
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 512
    vb.cpus = 1
  end
 # config.vm.define "mysqldb" do |mysql|
 #   mysql.vm.network "public_network"
 #   mysql.vm.provision "shell", inline: $script_mysql
  #  mysql.vm.provision "shell", 
 #     inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
 #   mysql.vm.provision "shell", 
 #     inline: "service mysql restart"
 # 
 #   mysql.vm.synced_folder "./configs", "/configs"
 #   mysql.vm.synced_folder ".", "/vagrant", disabled: true
#
#
 # end

  config.vm.define "phpweb" do |phpweb|
    phpweb.vm.network "forwarded_port", guest: 8888, host: 8888
    phpweb.vm.network "public_network"
    phpweb.vm.provision "shell", 
      inline: "apt-get update && apt-get install -y puppet"

    phpweb.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "./configs"
      puppet.manifest_file = "lamp.pp"

      config.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 2
        vb.name = "Servidor PHP7"
      end
    end

  end

  config.vm.define "mysqlserver" do |mysqlserver|
    mysqlserver.vm.network "public_network"
    mysqlserver.vm.provision "shell", inline: "cat /vagrant/configs/p_key.pub >> .ssh/authorized_keys"
  end

  config.vm.define "ansible" do |ansible|
    ansible.vm.network "public_network"
    ansible.vm.provision "shell", 
      inline: "cp /vagrant/p_key /home/vagrant && \
               chmod 600 /home/vagrant/p_key && \
               chown vagrant:vagrant /home/vagrant/p_key"
    ansible.vm.provision "shell", 
      inline: "sudo apt update && \
               sudo apt install -y software-properties-common && \
               sudo apt-add-repository --yes --update ppa:ansible/ansible && \
               sudo apt install -y ansible"
    ansible.vm.provision "shell",
      inline: "ansible-playbook -i /vagrant/configs/ansible/hosts /vagrant/configs/ansible/provision_mysqlserver.yml"
  end

  config.vm.define "docker" do |docker|
    docker.vm.provider "virtualbox" do |vb|
      vb.memory = 512
      vb.cpus = 1
      vb.name= "Docker"
    end

    docker.vm.provision "shell", inline: "apt-get update && apt-get install -y docker.io"
  end
end