require 'json'

VAGRANTFILE_API_VERSION = "2"

base_dir = File.expand_path(File.dirname(__FILE__))
cluster = JSON.parse(IO.read(File.join(base_dir,"cluster.json")))

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  cluster.each do |hostname, info|

    config.vm.define hostname do |cfg|

      cfg.vm.provider :virtualbox do |vb, override|
        override.vm.box = "trusty64"
        override.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
        override.vm.network :private_network, ip: "#{info["ip"]}"
        override.vm.hostname = hostname

        vb.name = 'vagrant-mesos-' + hostname
        vb.customize ["modifyvm", :id, "--memory", info["config"]["mem"], "--cpus", info["config"]["cpus"], "--hwvirtex", "on" ]
      end

      # make sure the mesos working dir is always present
      master_work_dir = "/var/run/mesos"
      if info["role"].eql? "master" then
        cfg.vm.provision :shell, :inline => "mkdir -p #{master_work_dir}"
      end

      # # provision nodes with ansible
      cfg.vm.provision :ansible do |ansible|
        # ansible parallell execution
        #ansible.limit = "all"

        # Role based execution
        case info["role"]
        when "master"
          ansible.playbook = "ansible/mesosphere.yml"
          ansible.extra_vars = {
            mesos_cluster_name: "vagrant-mesos-cluster",
            mesos_install_mode: "master",
            mesos_ip: "#{info["ip"]}",
            mesos_zookeepers: "zk://10.10.10.10:2181/mesos"
          }
        when "slave"
          ansible.playbook = "ansible/mesosphere.yml"
          ansible.extra_vars = {
            mesos_cluster_name: "vagrant-mesos-cluster",
            mesos_install_mode: "slave",
            mesos_ip: "#{info["ip"]}",
            mesos_masters: "zk://10.10.10.10:2181/mesos" # if zk connect to it, otherwise anounce other masters
          }
        when "marathon"
          ansible.playbook = "ansible/mesosphere.yml"
          ansible.extra_vars = {
            mesos_cluster_name: "vagrant-mesos-cluster",
            mesos_install_mode: "marathon",
            mesos_ip: "#{info["ip"]}",
            mesos_masters: "zk://10.10.10.10:2181/mesos", # if zk connect to it, otherwise anounce other masters
            marathon_zookeepers: "zk://10.10.10.10:2181/marathon"
          }
        when "zk"
          ansible.playbook = "ansible/zookeeper.yml"
          ansible.extra_vars = {
            myid: "1"
          }
        end

      end

    end

  end

end
