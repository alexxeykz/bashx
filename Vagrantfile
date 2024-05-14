# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :"bashx" => {
        :box_name => "generic/centos8",
        :ip_addr => '192.168.56.146'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.synced_folder "scripts", "/shfile"

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "2048"]
          end
          
          box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
            sudo dnf install nginx -y
            sudo systemctl enable nginx
            sudo systemctl start nginx
            sudo firewall-cmd --permanent --add-service=http
            sudo firewall-cmd --reload
            mv /shfile/full.sh /var/log/nginx
            mv /shfile/mail.sh /var/log/nginx
            mv /shfile/0mail.cron /etc/cron.hourly
            chmod +x /var/log/nginx/*.sh          

# yum update -y
          SHELL

      end
  end
end
