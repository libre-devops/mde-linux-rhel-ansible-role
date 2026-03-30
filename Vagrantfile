# -*- mode: ruby -*-
# vi: set ft=ruby :

timestamp = Time.now.strftime("%Y%m%d%H%M")
vm_count  = 1

Vagrant.configure("2") do |config|

  config.vm.box         = "generic/rhel9"
  config.vm.box_version = "4.3.12"
  config.ssh.insert_key = false
  
  (1..vm_count).each do |i|
    vm_name = "vagrant-vm-rhel-#{timestamp}-#{i}"

    config.vm.define "node#{i}" do |node|

      node.vm.hostname = vm_name
      node.vm.network "public_network", bridge: "Default Switch"

      node.vm.provider "hyperv" do |v|
        v.vmname    = vm_name
        v.memory    = 4096
        v.maxmemory = 8192
        v.cpus      = 2

        v.enable_virtualization_extensions = false

        v.vm_integration_services = {
          guest_service_interface: true,
          heartbeat:               true,
          key_value_pair_exchange: true,
          shutdown:                true,
          time_synchronization:    true,
          vss:                     false
        }
      end

      node.vm.provision "shell", privileged: true, inline: <<-SHELL
        set -eux
        echo "🔧 Bootstrapping #{vm_name}..."

        hostnamectl set-hostname #{vm_name}

        grep -q "#{vm_name}" /etc/hosts || echo "127.0.0.1 #{vm_name}" >> /etc/hosts
        grep -q "localhost"  /etc/hosts || echo "127.0.0.1 localhost"   >> /etc/hosts

        dnf update -y
        dnf install -y openssh-clients curl ca-certificates sudo python3

        echo "✅ #{vm_name} ready"
      SHELL

      node.vm.provision "file", source: "./keys/id_ed25519.pub", destination: "/tmp/id_ed25519.pub"

      node.vm.provision "shell", privileged: true, inline: <<-SHELL
        set -eux

        SSH_DIR="/home/vagrant/.ssh"
        AUTH_KEYS="$SSH_DIR/authorized_keys"

        mkdir -p $SSH_DIR
        chmod 700 $SSH_DIR

        cat /tmp/id_ed25519.pub >> $AUTH_KEYS

        chmod 600 $AUTH_KEYS
        chown -R vagrant:vagrant $SSH_DIR

        rm -f /tmp/id_ed25519.pub

        echo "🔑 SSH key added from file"
      SHELL

      node.vm.provision "shell", privileged: true, inline: <<-SHELL
        set -eux

        # Grab the first non-loopback IP
        VM_IP=$(hostname -I | awk '{print $1}')

        echo ""
        echo "================================================"
        echo "📋 inventory.ini"
        echo "================================================"
        echo "[rhel_mde_servers]"
        echo "#{vm_name} ansible_host=${VM_IP}"
        echo ""
        echo "[rhel_mde_servers:vars]"
        echo "ansible_user=vagrant"
        echo "================================================"
        echo ""
      SHELL
    end
  end
end