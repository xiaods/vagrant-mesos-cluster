VAGRANTFILE_API_VERSION = "2"

base_dir = File.expand_path(File.dirname(__FILE__))
cluster = {
  "mesos-master1" => { :ip => "100.0.10.11",  :cpus => 1, :mem => 1024 },
  "mesos-slave1"  => { :ip => "100.0.10.101", :cpus => 1, :mem => 1024 }
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine
    config.cache.enable :apt
  end

  cluster.each do |hostname, info|

    config.vm.define hostname do |cfg|

      cfg.vm.provider :virtualbox do |vb, override|
        override.vm.box = "trusty64"
        override.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
        override.vm.network :private_network, ip: "#{info[:ip]}"
        override.vm.hostname = hostname
        override.vm.synced_folder ".", "/vagrant", disabled: true

        vb.name = 'vagrant-mesos-' + hostname
        vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpus], "--hwvirtex", "on" ]
      end

      # provision nodes with ansible
      cfg.vm.provision :ansible do |ansible|
        ansible.verbose = "v"

        ansible.inventory_path = base_dir + "/inventory/vagrant"
        ansible.playbook = base_dir + "/cluster.yml"
        # ansible.skip_tags = [ "runtime" ]
        ansible.extra_vars = {
          vagrant_ip: "#{info[:ip]}"
        }
        ansible.limit = "#{info[:ip]}" # Ansible hosts are identified by ip
      end

    end

  end

end
