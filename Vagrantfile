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
        ansible.verbose = "v"

        if info["role"] == "zk" then
          ansible.playbook = "ansible/zookeeper.yml"
          ansible.extra_vars = {
            myid: "1"
          }
        else
          ansible.playbook = "ansible/mesosphere.yml"
          ansible.extra_vars = {
            mesos_mode: "#{info["role"]}",
            mesos_ip: "#{info["ip"]}",
            mesos_zk: "zk://10.10.10.10:2181/mesos"
          }

          case info["role"]
          when "master"
            ansible.extra_vars.merge!({
              mesos_options_master: {
                cluster: "vagrant-mesos-cluster",
                work_dir: "/var/run/mesos",
                quorum: 1
              }
            })
          when "slave"
            ansible.extra_vars.merge!({})
          when "marathon"
            ansible.extra_vars.merge!({
              marathon_zk: "zk://10.10.10.10:2181/marathon"
            })
          end
        end

      end

    end

  end

end
