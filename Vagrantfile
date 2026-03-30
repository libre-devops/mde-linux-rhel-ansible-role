# ============================================================
# 🧠 Dynamic VM Name
# ============================================================
timestamp = Time.now.strftime("%Y%m%d%H%M")
vm_name   = "vagrant-vm-rhel-#{timestamp}"

Vagrant.configure("2") do |config|

  # ============================================================
  # 📦 Base box
  # ============================================================
  config.vm.box = "generic/rhel9"
  config.vm.box_version = "4.3.12"
  config.vm.hostname = vm_name

  # ============================================================
  # 🔐 SSH
  # ============================================================
  config.ssh.insert_key = false

  # ============================================================
  # 🌐 Networking
  # ============================================================
  config.vm.network "public_network",
    bridge: "Default Switch"

  # ============================================================
  # 💻 Hyper-V
  # ============================================================
  config.vm.provider "hyperv" do |v|
    v.vmname = vm_name
    v.memory = 4096
    v.maxmemory = 8192
    v.cpus = 2

    v.enable_virtualization_extensions = false

    v.vm_integration_services = {
      guest_service_interface: true,
      heartbeat: true,
      key_value_pair_exchange: true,
      shutdown: true,
      time_synchronization: true,
      vss: false
    }
  end

  # ============================================================
  # 🧪 Bootstrap
  # ============================================================
  config.vm.provision "shell", privileged: true, inline: <<-SHELL
    echo "🔧 Bootstrapping RHEL VM..."

    # Set hostname properly
    hostnamectl set-hostname #{vm_name}

    # Ensure /etc/hosts is clean + consistent
    if ! grep -q "#{vm_name}" /etc/hosts; then
      echo "127.0.0.1   #{vm_name}" >> /etc/hosts
    fi

    # Optional: ensure localhost mapping still exists
    if ! grep -q "localhost" /etc/hosts; then
      echo "127.0.0.1   localhost" >> /etc/hosts
    fi

    # Packages
    dnf -y update
    dnf -y install python3

    alternatives --set python /usr/bin/python3 || true

    echo "✅ Hostname set to: #{vm_name}"
    hostname

    echo "✅ Bootstrap complete"
  SHELL
end