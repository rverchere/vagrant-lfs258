# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = true

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 1
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y
    apt-get install -y vim

    apt-get install -y docker.io
    systemctl enable docker

    echo "deb  http://apt.kubernetes.io/  kubernetes-xenial  main" > /etc/apt/sources.list.d/kubernetes.list
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    apt-get update
    apt-get install -y kubeadm=1.18.1-00 kubelet=1.18.1-00 kubectl=1.18.1-00
    apt-mark hold kubelet kubeadm kubectl

    echo "192.168.56.10 k8smaster" >> /etc/hosts
  SHELL

  config.vm.define "k8smaster", primary: true do |k8smaster|
    k8smaster.vm.hostname = "k8smaster"
    k8smaster.vm.network :private_network, ip: "192.168.56.10"
    k8smaster.vm.provider "virtualbox" do |vmaster|
      vmaster.memory = 4096
      vmaster.cpus = 2
    end
    k8smaster.vm.provision "shell", inline: <<-SHELL
      kubeadm init --config=/vagrant/kubeadm-config.yaml --upload-certs | tee /home/vagrant/kubeadm-init.out

      sudo -u vagrant mkdir -p /home/vagrant/.kube
      sudo cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
      chown vagrant: /home/vagrant/.kube/config

      sudo -u vagrant kubectl create -f /vagrant/calico-tigera-operator.yaml
      sudo -u vagrant kubectl create -f /vagrant/calico-custom-resources.yaml

      sudo apt-get install bash-completion -y
      sudo -u vagrant echo "source <(kubectl completion bash)" >> /home/vagrant/.bashrc

      sudo -u vagrant kubectl taint nodes --all node-role.kubernetes.io/master-
   SHELL
  end

  config.vm.define "k8sworker1" do |k8sworker1|
    k8sworker1.vm.hostname = "k8sworker1"
    k8sworker1.vm.network :private_network, ip: "192.168.56.11"
  end
  config.vm.define "k8sworker2" do |k8sworker2|
    k8sworker2.vm.hostname = "k8sworker2"
    k8sworker2.vm.network :private_network, ip: "192.168.56.12"
  end

end
