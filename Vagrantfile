Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  #config.vm.provision "shell", path: "k8s_setup.sh"
  config.vm.synced_folder ".", "/vagrant", type: "nfs"

  config.vm.define "k1" do |k1|
    k1.vm.hostname = "k1"
    k1.vm.provision "shell", path: "k8s_controlpane.sh"
    k1.vm.provider :libvirt do |domain|
      domain.memory = 2048
      domain.cpus = 2
    end
  end

  config.vm.define "k2" do |k2|
    k2.vm.hostname = "k2"
    k2.vm.provision "shell", path: "k8s_compute.sh"
    k2.vm.provider :libvirt do |domain|
      domain.memory = 2048
      domain.cpus = 2
    end
  end

  config.vm.define "k3" do |k3|
    k3.vm.hostname = "k3"
    k3.vm.provision "shell", path: "k8s_compute.sh"
    k3.vm.provider :libvirt do |domain|
      domain.memory = 2048
      domain.cpus = 2
    end
  end

end
