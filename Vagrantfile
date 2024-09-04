# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
if Vagrant.has_plugin?("vagrant-vbguest")
 config.vbguest.auto_update = false
 config.vbguest.no_remote = true
 end 
 config.vm.box = "bento/ubuntu-22.04" 
 config.vm.provider "virtualbox" do |v| 
 v.memory = 1024
 v.cpus = 2
 end 
 config.vm.define "sysd" do |sysd| 
 sysd.vm.network "private_network", ip: "192.168.56.16",  virtualbox__intnet: "net1"
 sysd.vm.hostname = "sysd"
 config.vm.provision "file", source: "/home/nur/hw-9-sysd/scripts/", destination: "/tmp/"
 config.vm.provision "shell", inline: <<-SHELL
 apt update
 apt install spawn-fcgi php php-cgi php-cli apache2 libapache2-mod-fcgid nginx -y
 cp /tmp/scripts/watchlog /etc/default
 cp /tmp/scripts/watchlog.log /var/log
 cp /tmp/scripts/watchlog.sh /opt
 chmod +x /opt/watchlog.sh
 cp /tmp/scripts/watchlog.service /etc/systemd/system
 cp /tmp/scripts/watchlog.timer /etc/systemd/system
 systemctl start watchlog.timer
 systemctl start watchlog.service
 mkdir /etc/spawn-fcgi
 cp /tmp/scripts/fcgi.conf /etc/spawn-fcgi
 cp /tmp/scripts/spawn-fcgi.service /etc/systemd/system
 systemctl start spawn-fcgi
 systemctl status spawn-fcgi
 cp /tmp/scripts/nginx@.service /etc/systemd/system
 cp /tmp/scripts/nginx-first.conf /etc/nginx
 cp /tmp/scripts/nginx-second.conf /etc/nginx
 systemctl start nginx@first
 systemctl start nginx@second

 SHELL
 end 
 
end 
