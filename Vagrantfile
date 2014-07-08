VAGRANTFILE_API_VERSION = "2"

cluster = {
  zk: {
    nodes: 1,
    cpus: 1,
    mem: 256,
    ip: "10.10.10.",
    prefix: "zk"
  },
  master: {
    nodes: 1,
    cpus: 1,
    mem: 256,
    ip: "10.10.20.",
    prefix: "master"
  },
  slave: {
    nodes: 1,
    cpus: 2,
    mem: 512,
    ip: "10.10.30.",
    prefix: "slave"
  }
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  cluster.each do |role, info|
    info[:nodes].times do |i|
      hostname = "#{info[:prefix]}#{i}"

      config.vm.define hostname do |cfg|
        cfg.vm.provider :virtualbox do |vb, override|
          override.vm.box = "trusty64"
          override.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
          override.vm.network :private_network, ip: "#{info[:ip]}#{10+i}"
          override.vm.hostname = hostname

          vb.name = 'vagrant-mesos-' + hostname
          vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpus], "--hwvirtex", "on" ]
        end

        # mesos-master doesn't create its work_dir.
        master_work_dir = "/var/run/mesos"
        if role.eql? "master" then
          cfg.vm.provision :shell, :inline => "mkdir -p #{master_work_dir}"
        end

        # provision nodes with ansible
        cfg.vm.provision :ansible do |ansible|
          # ansible parallell execution
          #ansible.limit = "all"

          # Role based execution
          case role
          when :master
            ansible.playbook = "ansible/mesosphere.yml"
            ansible.extra_vars = {
              mesos_cluster_name: "vagrant-mesos-cluster",
              mesos_install_mode: "master",
              mesos_ip: "#{info[:ip]}#{10+i}",
              mesos_zookeepers: "zk://10.10.10.10:2181/mesos"
            }
          when :slave
            ansible.playbook = "ansible/mesosphere.yml"
            ansible.extra_vars = {
              mesos_cluster_name: "vagrant-mesos-cluster",
              mesos_install_mode: "slave",
              mesos_ip: "#{info[:ip]}#{10+i}",
              mesos_masters: "zk://10.10.10.10:2181/mesos" # if zk connect to it, otherwise anounce other masters
            }
          when :zk
            ansible.playbook = "ansible/zookeeper.yml"
            ansible.extra_vars = {
              myid: "#{i+1}"
            }
          end

        end
      end
    end
  end


  # config.vm.box = "trusty64"
  # config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
  # config.vm.network :forwarded_port, guest: 8080, host: 8080
  # config.vm.network :forwarded_port, guest: 5050, host: 5050
  # config.vm.network :private_network, ip: "10.10.1.10"
  # # config.vm.network :public_network
  # # config.vm.synced_folder "../data", "/vagrant_data"

  # # Provider-specific configuration
  # config.vm.provider :virtualbox do |vb|
  #   vb.customize ["modifyvm", :id, "--memory", "512"]
  # end

  # config.vm.provision "shell", path: "provision.sh"
end
