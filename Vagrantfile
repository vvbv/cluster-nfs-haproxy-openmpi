# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.vm.define :master do |master|
        
        master.vm.box = "debian/stretch64"

        master.vm.provider :virtualbox do |vbox|

            vbox.name = "cluster_master"
            vbox.memory = 256
            vbox.cpus = 1
            vbox.customize [ "modifyvm", :id, "--nic2", "hostonly" ]
            vbox.customize [ "modifyvm", :id, "--hostonlyadapter2", "vboxnet0" ]

        end

        master.vm.network "private_network", :ip => '10.11.13.3', :adapter => 2
        
        master.vm.provision "instalación de nfs", type: "shell", inline: <<-SCRIPT
            apt update
            apt install nfs-kernel-server -y
        SCRIPT

        master.vm.provision "instalación de haproxy y curl", type: "shell", inline: <<-SCRIPT
            apt install haproxy curl -y
        SCRIPT

        master.vm.provision "instalación de OpenMPI", type: "shell", inline: <<-SCRIPT
            apt install openmpi-bin openmpi-common libopenmpi2 libopenmpi-dev -y
        SCRIPT

        master.vm.provision "preparación de directorios", type: "shell", inline: <<-SCRIPT
            mkdir -p /export/shared
            mkdir /shared
            chmod 777 /{export,shared} && sudo chmod 777 /export/*
            mount --bind /shared /export/shared
        SCRIPT

        master.vm.provision "configuración del servidor nfs", type: "shell", inline: <<-SCRIPT
            echo "/export/shared *(rw,fsid=0,insecure,no_root_squash,no_subtree_check,sync)" >> /etc/exports
            systemctl restart nfs-kernel-server
            systemctl enable nfs-kernel-server
        SCRIPT

        master.vm.provision "configuración del balanceador de carga haproxy", type: "shell", inline: <<-SCRIPT
            echo "frontend haproxy_in" >> /etc/haproxy/haproxy.cfg 
            echo "    bind *:80" >> /etc/haproxy/haproxy.cfg 
            echo "    default_backend haproxy_http" >> /etc/haproxy/haproxy.cfg 
            echo "backend haproxy_http" >> /etc/haproxy/haproxy.cfg 
            echo "    balance roundrobin" >> /etc/haproxy/haproxy.cfg 
            echo "    mode http" >> /etc/haproxy/haproxy.cfg 
            echo "    server node1 10.11.13.4:80 check" >> /etc/haproxy/haproxy.cfg 
            echo "    server node2 10.11.13.5:80 check" >> /etc/haproxy/haproxy.cfg 
            systemctl restart haproxy
            systemctl enable haproxy
        SCRIPT

        master.vm.provision "configuración de OpenMPI", type: "shell", inline: <<-SCRIPT
            rm -rf /root/.ssh/*
            ssh-keygen -b 2048 -t rsa -f /root/.ssh/id_rsa -q -N ""
            cp /root/.ssh/id_rsa.pub /export/shared/authorized_keys
        SCRIPT

        master.vm.provision "configuración de hostname", type: "shell", inline: <<-SCRIPT
            echo "cluster_master" > /etc/hostname
        SCRIPT

    end

    (1..2).each do |i|

        config.vm.define "node#{i}" do |node|
        
            node.vm.box = "debian/stretch64"
    
            node.vm.provider :virtualbox do |vbox|
    
                vbox.name = "cluster_node_#{i}"
                vbox.memory = 256
                vbox.cpus = 1
                vbox.customize [ "modifyvm", :id, "--nic2", "hostonly" ]
                vbox.customize [ "modifyvm", :id, "--hostonlyadapter2", "vboxnet0" ]
    
            end

            node.vm.network "private_network", :ip => "10.11.13.#{i+3}", :adapter => 2
        
            node.vm.provision "instalación de nfs", type: "shell", inline: <<-SCRIPT
                apt update
                apt install nfs-common -y
            SCRIPT

            node.vm.provision "instalación de apache", type: "shell", inline: <<-SCRIPT
                apt install apache2 -y
            SCRIPT

            node.vm.provision "configuración de apache", type: "shell", inline: <<-SCRIPT
                systemctl restart apache2
                systemctl enable apache2
            SCRIPT

            node.vm.provision "instalación de OpenMPI", type: "shell", inline: <<-SCRIPT
                apt install openmpi-bin openmpi-common libopenmpi2 libopenmpi-dev -y
            SCRIPT

            node.vm.provision "configuración de sitio base", type: "shell", inline: <<-SCRIPT
                echo "Respuesta desde node#{i}" > /var/www/html/index.html
            SCRIPT
    
            node.vm.provision "creación de los directorios", type: "shell", inline: <<-SCRIPT
                mkdir /shared
                chmod 777 /shared
            SCRIPT

            node.vm.provision "configuración del cliente nfs", type: "shell", inline: <<-SCRIPT
                echo "10.11.13.3:/export/shared  /shared  nfs  auto  0  0" >> /etc/fstab
            SCRIPT

            node.vm.provision "conexión con cluster master", type: "shell", inline: <<-SCRIPT
                mount -a
            SCRIPT

            node.vm.provision "configuración de OpenMPI", type: "shell", inline: <<-SCRIPT
                mkdir -p /root/.ssh/
                chmod 700 /root/.ssh/
                cp /shared/authorized_keys /root/.ssh/ 
            SCRIPT

            node.vm.provision "configuración de hostname", type: "shell", inline: <<-SCRIPT
                echo "cluster_node_#{i}" > /etc/hostname
            SCRIPT
    
        end

    end

end
