Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |v|
    v.memory = 1024
    v.cpus = 1
  end

  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.provision :shell, privileged: true, inline: $install_common_tools

  config.vm.define :master do |master|
    master.vm.box = "ubuntu20.04"
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "192.168.56.100"
    #master.vm.provider :virtualbox do |vb|
    #  vb.customize ["modifyvm", :id, "--memory", "4096"]
    #  vb.customize ["modifyvm", :id, "--cpus", "4"]
    #end
  end

  %w{worker1}.each_with_index do |name, i|
    config.vm.define name do |worker|
      worker.vm.box = "ubuntu20.04"
      worker.vm.hostname = name
      worker.vm.network :private_network, ip: "192.168.56.#{i + 101}"
      #worker.vm.provider :virtualbox do |vb|
      #  vb.customize ["modifyvm", :id, "--memory", "3072"]
      #  vb.customize ["modifyvm", :id, "--cpus", "3"]
      #end
    end
  end

end

$install_common_tools = <<-SCRIPT

# suppress hash sum mismatch issue
mkdir /etc/gcrypt
echo all >> /etc/gcrypt/hwf.deny

# replace with aliyun apt mirror
cp /etc/apt/sources.list /etc/apt/sources.list.bak
cat > /etc/apt/sources.list <<EOF
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
EOF

apt update -y

# bridged traffic to iptables is enabled for kube-router.
cat >> /etc/ufw/sysctl.conf <<EOF
net/bridge/bridge-nf-call-ip6tables = 1
net/bridge/bridge-nf-call-iptables = 1
net/bridge/bridge-nf-call-arptables = 1
EOF

# disable swap
swapoff -a
sed -i '/swap/d' /etc/fstab

SCRIPT

$apt_install = <<-SCRIPT

# install required package
sudo apt install -y apt-transport-https ca-certificates curl
sudo apt install ipset socat

# install docker-ce
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo echo deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable | sudo tee /etc/apt/sources.list.d/docker.list

sudo apt -y update
sudo apt -y install docker-ce

SCRIPT

