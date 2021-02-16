# -*- mode: ruby -*-
# vi: set ft=ruby :

NODE_IP         = "10.0.0.10"
GO_VERSION      = "1.15.8"
ETCD_VERSION    = "3.3.13"
DOCKER_VERSION  = ""
GITHUB_USERNAME = ENV["GITHUB_USERNAME"]

if GITHUB_USERNAME == nil || GITHUB_USERNAME == ""
  abort("The Github username is required. Export the environment variable GITHUB_USERNAME")
end

# Install the same Go version you have in your host system
my_go_version = `go version`.match /.* go(.*) .*/
if my_go_version.length() > 1 || my_go_version[1] != ""
  GO_VERSION = my_go_version[1]
else
  puts "[WARN] Go was not found on your system, it's recommended to install it. Go version: #{my_go_version}"
end

if ENV["FULLNAME"] == nil
  FULLNAME = `git config --get user.name`
end
if ENV["EMAIL"] == nil
  EMAIL = `git config --get user.email`
end

H1 = <<-SHELL
  h1(){
    H1="\x1B[93;1m==[ "
    H0=" ]==\x1B[0m"
    echo -e $H1$1$H0"\n"
  }
SHELL

ERR = <<-SHELL
  err(){
    E1="\x1B[91;1m==[ ERROR: "
    E0=" ]==\x1B[0m"
    echo -e $E1$1$E0"\n"
    exit 1
  }
SHELL

TEST_PROVISIONING = <<-SHELL
  go version
  docker version
  etcd version
  etcdctl member list

  curl --version
  jq --version
  python3 --version
  pip3 --version

  ls -l shared_folder/k8s.io/kubernetes
  ls -l shared_folder/k8s.io/website
  ls -l shared_folder/k8s.io/test-infra

  git config -l | cat
SHELL

PROVISION_NODE = <<-SHELL
  #{H1}
  #{ERR}

  [[ -z "#{GITHUB_USERNAME}" ]] && err "The Github username is required. Export the environment variable GITHUB_USERNAME."

  h1 "Upgrading system and installing dependencies"
  apt-get update && apt-get upgrade -y
  apt-get install -y \
    apt-transport-https \
    curl \
    ca-certificates \
    build-essential \
    dkms \
    linux-headers-$(uname -r) \
    gnupg-agent \
    software-properties-common

  h1 "Installing Go version #{GO_VERSION}"
  curl -fsSL -O https://storage.googleapis.com/golang/go#{GO_VERSION}.linux-amd64.tar.gz
  tar -C /usr/local -xzf go#{GO_VERSION}.linux-amd64.tar.gz
  grep -q local/go/bin /etc/profile || echo "export PATH=/usr/local/go/bin:${PATH}" >> /etc/profile
  source /etc/profile
  go version

  if [[ -n "#{DOCKER_VERSION}" ]]; then
    DOCKER_VERSION="=#{DOCKER_VERSION}"
    h1 "Installing Docker version #{DOCKER_VERSION}"
  else
    h1 "Installing Docker latest version"
  fi
  apt-get update
  apt-get remove -y docker docker-engine docker.io containerd runc
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt-get update

  apt-get install -y \
    docker-ce${DOCKER_VERSION} \
    docker-ce-cli${DOCKER_VERSION} \
    containerd.io
  usermod -aG docker vagrant

  h1 "Installing Build Tools"
  apt-get install -y \
    build-essential \
    jq \
    python3-pip
  pip3 install pyyaml

  h1 "Setup Github from github.com/#{GITHUB_USERNAME}"
  KUBE_ROOT=/home/vagrant/shared_folder/k8s.io
  mkdir -p ${KUBE_ROOT}
  cd ${KUBE_ROOT}

  setup_project() {
    project=$1

    if [[ ! -e $project ]]; then
      git clone https://github.com/#{GITHUB_USERNAME}/${project}.git
      [[ $? -ne 0 ]] && err "Did you fork the Kubernetes project ${project}?"
    fi

    cd ${project}
    git status
    [[ $? -ne 0 ]] && exit 1
    cd -
  }

  setup_project kubernetes
  setup_project website
  setup_project test-infra

  cd ${KUBE_ROOT}/kubernetes
  git config --global user.name "#{FULLNAME}"
  git config --global user.email "#{EMAIL}"
  git config --global push.default simple
  git config credential.helper store
  git remote add upstream https://github.com/kubernetes/kubernetes.git

  git config -l | cat

  h1 "Installing etcd"
  ${KUBE_ROOT}/kubernetes/hack/install-etcd.sh
  grep -q third_party/etcd /etc/profile || echo "export PATH=${KUBE_ROOT}/kubernetes/third_party/etcd:${PATH}" >> /etc/profile
  source /etc/profile

  etcd --version
  etcdctl member list

  echo "#{TEST_PROVISIONING}" > /home/vagrant/test.sh
  chmod +x /home/vagrant/test.sh
  chown vagrant /home/vagrant/test.sh

  h1 "Your Github repositories are located in ./shared_folders/k8s.io"

  h1 "Find issues at https://github.com/kubernetes/kubernetes/issues"
SHELL

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  config.vm.provision "shell", inline: PROVISION_NODE
  config.vm.synced_folder "./shared_folder", "/home/vagrant/shared_folder"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "8192"
    vb.cpus = "2"
    vb.linked_clone = true
    vb.gui = false
  end

  config.vm.define "kude" do |m|
    m.vm.provider "virtualbox" do |vb|
      name = "kube-dev-env"
    end
    m.vm.hostname = "kude"
    m.vm.network "private_network",
      ip: NODE_IP,
      netmask: "255.255.255.0",
      auto_config: true
  end
end
