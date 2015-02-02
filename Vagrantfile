# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Check if this is Windows
  is_windows = (RbConfig::CONFIG['host_os'] =~ /mswin|mingw|cygwin/)

  # Look for project variables file.
  if !(File.exists?(File.dirname(__FILE__) + "/vars.project.yml"))
    raise NoVarsException
  end

  # Load the yml files
  require 'yaml'
  vars = YAML.load_file(File.dirname(__FILE__) + "/vars.global.yml")
  vars.merge!(YAML.load_file(File.dirname(__FILE__) + "/vars.project.yml"))

  # Config the VM
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = vars['server_hostname']

  # Sets IP of the guest machine and allows it to connect to the internet.
  config.vm.network :private_network, ip:  vars['vansible_ip']
  config.ssh.forward_agent = true

  # Read this user's host machine's public ssh key to pass to ansible.
  if !(File.exists?("#{Dir.home}/.ssh/id_rsa.pub"))
    if !(File.exists?("#{Dir.home}/.ssh/id_dsa.pub"))
      raise NoSshKeyException
    end
  end
  if !(File.exists?("#{Dir.home}/.ssh/id_rsa.pub"))
    ssh_public_key = IO.read("#{Dir.home}/.ssh/id_dsa.pub").strip!
  else
    ssh_public_key = IO.read("#{Dir.home}/.ssh/id_rsa.pub").strip!
  end

  # If ansible is installed on the host, we can use config.vm.provision.
  # If it is not, we use shell provisioner to run
  # See https://github.com/mitchellh/vagrant/issues/2103
  has_ansible = `which ansible`.to_s.strip.length != 0
  if has_ansible
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = vars['vansible_playbook']
      ansible.extra_vars = {
        ansible_ssh_user: 'vagrant',
        authorized_keys: ssh_public_key
      }
      ansible.sudo = true
    end
  else
    # If local ansible is not found, install it in the guest and run the playbook there.
    raise NoAnsibleException
  end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", vars['vansible_memory']]
  end

end

##
# Our Exceptions
#
class NoVarsException < Vagrant::Errors::VagrantError
  error_message('Project variables file not found. Copy vars.project.example.yml to vars.project.yml, edit to match your project, then try again.')
end
class NoAnsibleException < Vagrant::Errors::VagrantError
  error_message("Ansible isn't installed on your system. On debian based systems, it can be installed with:
      apt-get install python-software-properties -y
      add-apt-repository ppa:rquillo/ansible
      apt-get update
      apt-get install ansible -y

    On Mac, it can be installed using Homebrew like this:
      brew install ansible")
end
