# Set the default provider to QEMU/KVM
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt' unless ENV['VAGRANT_DEFAULT_PROVIDER']

# All Vagrant configuration is done below.
Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  
  # Define methods to calculate memory and CPUs
  def calculate_memory
    # Example: Set memory to half of the total available memory on the host (in MB)
    total_memory_mb = `free -m | awk '/^Mem:/{print $2}'`.to_i
    (total_memory_mb / 2).clamp(512, total_memory_mb).to_i # Ensure at least 512 MB and not more than total memory
  end

  def calculate_cpus
    # Example: Set CPUs to half of the total available CPUs on the host
    total_cpus = `nproc`.to_i
    (total_cpus / 2).clamp(1, total_cpus).to_i # Ensure at least 1 CPU and not more than total CPUs
  end

  def configure_vm(config, name, box, playbook, hostname, ports)
    config.vm.define name do |vm_config|
      vm_config.vm.box = box
      vm_config.vm.hostname = hostname
      vm_config.vm.network "private_network", type: "dhcp"

      ports.each do |port|
        vm_config.vm.network "forwarded_port", guest: port, host: port
      end

      vm_config.vm.provider "libvirt" do |lv|
        # Dynamically set memory and CPUs based on host specs
        lv.memory = calculate_memory
        lv.cpus = calculate_cpus

        # Use VirtIO settings for optimal performance
        lv.graphics_type = "spice"
        lv.video_type = "qxl"
        lv.nic_model_type = "virtio"
        lv.disk_bus = "virtio"
        lv.driver = "kvm"
        lv.uri = 'qemu:///system'
      end

      # Uncomment to enable Ansible provisioning
      vm_config.vm.provision "ansible" do |ansible|
        ansible.playbook = playbook
         ansible.inventory_path = ".vagrant/provisioners/ansible/inventory"  # Point to Vagrant's inventory
      end
    end
  end

  # Ports to be forwarded
  ports = [3000, 3080, 3081, 12345]

  # Configure Alpine VMs
  configure_vm(config, "alpine-docker", "roboxes/alpine319", "ansible/hosts/alpine-docker.yaml", "alpine-docker.local", ports)
  configure_vm(config, "alpine-podman", "roboxes/alpine319", "ansible/hosts/alpine-podman.yaml", "alpine-podman.local", ports)

  # Configure Fedora VMs
  configure_vm(config, "fedora-docker", "fedora/41-cloud-base", "ansible/hosts/fedora-docker.yaml", "fedora-docker.local", ports)
  configure_vm(config, "fedora-podman", "fedora/41-cloud-base", "ansible/hosts/fedora-podman.yaml", "fedora-podman.local", ports)
  # Uncomment to configure Debian VMs if needed
  # configure_vm(config, "debian-docker", "generic/debian12", "playbooks/debian-docker.yml", "debian-docker.local", ports)
  # configure_vm(config, "debian-podman", "generic/debian12", "playbooks/debian-podman.yml", "debian-podman.local", ports)

  config.vm.synced_folder "./compose", "/app", type: "rsync"
end

