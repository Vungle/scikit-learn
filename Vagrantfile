#-*- mode: ruby -*-
# vi: set ft=ruby :

require 'etc'

VAGRANTFILE_API_VERSION = "2"

USER = Etc.getpwuid(Process.uid).name

BASIC_PROVISION_SCRIPT = <<SHELL
apt-get update
apt-get install -y \
    build-essential python-dev python-setuptools python-pip \
    git \
    python-numpy python-scipy \
    libatlas-dev libatlas3gf-base

cat >> /etc/pip.conf <<EOF
[global]
index-url = http://pypi.vungle.com:3000/simple
EOF

pip install virtualenv
virtualenv /opt/scikit_learn/venv
. /opt/scikit_learn/venv/bin/activate

pip install --trusted-host pypi.vungle.com numpy
pip install --trusted-host pypi.vungle.com scipy 
pip install --trusted-host pypi.vungle.com cython
pip install --trusted-host pypi.vungle.com nose

cat > /home/vagrant/.pypirc <<EOF
[distutils]
index-servers = vungle

[vungle]
repository = http://pypi.vungle.com:3000/
#username = [username]
#password = [password]
EOF

cat >> /home/vagrant/.profile <<EOF
. /opt/scikit_learn/venv/bin/activate
EOF
SHELL

def get_memory()
  case RbConfig::CONFIG["host_os"]
    when /darwin/
      `sysctl hw.memsize`.split()[-1].to_i / 1024 / 1024
    else
      # TODO probably it's better to throw an error here (or at least a
      #      warning)
      8196  # if can't detect, be conservative
  end
end

def get_cpus()
  case RbConfig::CONFIG["host_os"]
    when /darwin/
      `sysctl -n hw.ncpu`.to_i
    else
      # if can't detect, default to 1
      1
  end
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/precise64"
  config.vm.hostname = "sklearn"
  config.vm.network "private_network", ip: "10.10.0.11"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder ".", "/opt/scikit_learn/repo"

  config.vm.provision "shell", inline: BASIC_PROVISION_SCRIPT

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id,
                  # reserve 65% of host memory for this machine
                  "--memory", (get_memory() * 0.65).to_i.to_s,
                  "--cpus", get_cpus().to_s,
                  # allowed to use up to 90% of CPU resources
                  "--cpuexecutioncap", "90"]
  end
end
