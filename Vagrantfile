# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
GROUP_VARS_FILE = "./ansible/group_vars/all"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Make sure user has copied the template file
  if not File.exists? GROUP_VARS_FILE
    abort "\n### Error building vagrant-dash: Unable to find #{GROUP_VARS_FILE}\n\n  See the 'Getting Started' section of the README.md file\n\n"
  end
  
  begin
    group_vars = YAML::load(File.open(GROUP_VARS_FILE))
  rescue
    abort "\nError reading #{GROUP_VARS_FILE}, make sure it is valid YAML\n\n"
  end
  # Make sure user has customized the template file.
  # Use ruby exception handling to catch errors

  # Set the name
  config.vm.define "vagrant-dash"
  config.vm.box = "dryad"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider :aws do |aws, override|
    override.vm.box = "dummy"
    aws.access_key_id = ENV["DRYAD_AWS_ACCESS_KEY_ID"]
    aws.secret_access_key = ENV["DRYAD_AWS_SECRET_ACCESS_KEY"]
    aws.keypair_name = ENV["DRYAD_AWS_KEYPAIR_NAME"]
    aws.tags = {
       'Name' => group_vars['machine_name']
    }
    # From http://cloud-images.ubuntu.com/locator/ec2/
    # us-east-1	xenial	16.04 LTS	amd64	hvm-ssd	20180109	ami-41e0b93b
    aws.ami = "ami-41e0b93b"
    aws.instance_type = "t2.small"
    aws.security_groups = ['default', 'dryad-privileges']
    aws.block_device_mapping = [{ 'DeviceName' => '/dev/sda1', 'Ebs.VolumeSize' => 50 }]
    override.ssh.username = "ubuntu"
    override.ssh.private_key_path = ENV["DRYAD_AWS_PRIVATEKEY_PATH"]
  end
  config.vm.synced_folder ".", "/ubuntu", type: "rsync",
    rsync__exclude: [".git/"]

  #
  # View the documentation for the provider you're using for more
  # information on available options.
  config.vm.provision "ansible" do |ansible|
    ansible.groups = {
      "dryad_servers" => ["vagrant-dash"]
    }
    ansible.limit = 'all'
    ansible.playbook = "./ansible/setup.yml"
    ansible.sudo = true
    ansible.host_key_checking = false
    ansible.verbose = 'v'
    ansible.galaxy_role_file = './ansible/roles/requirements.yml'
    ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
  end
end
