Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |vbox|
    vbox.memory = 2048
    vbox.cpus = 2
  end

  config.vm.synced_folder ".", "/vagrant"

  config.vm.box = "ubuntu/focal64"
  config.vm.provision "ansible_local" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.extra_vars = {
      owner: "vagrant",
      domain_name: "testing.com" # Caddy won't be happy but it's expected
    }
    ansible.playbook = "homemedia-setup.yml"
    ansible.become = true
    ansible.become_user = "root"

    # Would have preferred from apt but... https://github.com/hashicorp/vagrant/issues/12204
    ansible.install_mode = "pip_args_only"
    ansible.pip_install_cmd = "sudo apt-get install -y python3-pip python-is-python3 haveged && sudo ln -s -f /usr/bin/pip3 /usr/bin/pip"
    ansible.pip_args = "ansible==5.8.0"
  end
end
