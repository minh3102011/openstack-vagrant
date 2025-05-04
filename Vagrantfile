Vagrant.configure("2") do |config|
    #configvmbox ubutnu22.04
    config.vm.box = "bento/ubuntu-22.04"  # Dùng box Ubuntu 22.04
    config.vm.hostname = "Openstack"     # Đặt hostname là Openstack
    config.ssh.username = "vagrant"       # Tên người dùng SSH
    config.ssh.password = "vagrant"       # Mật khẩu SSH
    config.vm.provider "vmware_desktop" do |vmware|
      vmware.gui = false                     # Chạy chế độ không có GUI
      vmware.memory = 4096                    # RAM 4GB
      vmware.cpus = 2                          # 2 CPU
      vmware.vmx["displayName"] = "Openstack" # Đặt tên máy ảo trên VMware
    end
    config.vm.network "public_network", bridge: "VMware Bridge Adapter"
  
    config.vm.provision "shell", inline: <<-SHELL
      # Cập nhật và cài đặt git
      sudo apt-get update && sudo apt-get install -y git
      # Thêm user stack
      sudo useradd -s /bin/bash -d /opt/stack -m stack
      sudo chmod +x /opt/stack
      echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
      HOST_IP=$(ip route get 1 | awk '{print $7; exit}')
      sudo -u stack -i bash <<EOF
  cd /opt/stack
  git clone https://opendev.org/openstack/devstack
  cd devstack
cat > local.conf <<EOL
[[local|localrc]]
HOST_IP=$HOST_IP
ADMIN_PASSWORD=admin
DATABASE_PASSWORD=admin
RABBIT_PASSWORD=admin
SERVICE_PASSWORD=admin
FORCE=yes
EOL
  ./stack.sh
  # Khởi động OpenStack
  source /opt/stack/devstack/openrc admin admin
  export OS_CLOUD=devstack-admin
  export PATH=$PATH:/opt/stack/bin
  # Tạo ảnh Ubuntu Cloud
  wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
  openstack image create "Ubuntu-22.04" --file jammy-server-cloudimg-amd64.img --disk-format qcow2 --container-format bare --public
  # Cấu hình mạng, flavor, keypair, instance
  openstack network create demo-net
  openstack subnet create --network demo-net --subnet-range 192.168.10.0/24 demo-subnet
  openstack flavor create --vcpus 1 --ram 2048 --disk 10 demo-flavor
  ssh-keygen -t rsa -b 2048 -f ~/demo-key -N ""
  openstack keypair create --public-key ~/demo-key.pub demo-key
  openstack server create --image "Ubuntu-22.04" --flavor m1.small --network demo-net --key-name demo-key demo-instance
  openstack volume create --size 10 demo-volume
  openstack server add volume demo-instance demo-volume
  EOF
      sudo systemctl restart apache2
    SHELL
  end
