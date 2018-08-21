vagrant-dryad
=============

Vagrant and Ansible config for building a Dryad VM

Uses Ubuntu 16.04 64-bit, Ansible, MySQL, Java8, Ruby

See also [Dryad installation](https://github.com/CDL-Dryad/dryad/blob/master/documentation/dryad_install.md) for the details of the process that is automated by this codebase.

## Requirements

These applications must be installed on your host machine.  They will be used to build and run a virtual machine for Dryad.

1. [Vagrant](http://vagrantup.com) (version 2.0 or higher)
2. [Ansible](http://ansible.com) (version 2.0 or higher)
3. [VirtualBox](http://virtualbox.org) (not required if you will be hosting the VM on the Amazon AWS cloud)

Vagrant and VirtualBox installation packages can be downloaded from their respective websites.  Ansible is a Python package with [many ways to install](http://docs.ansible.com/intro_installation.html).  One method is to use [Homebrew](http://brew.sh) (which, in turn, requires ruby) and simply `brew install ansible`.

## Getting started

You will need to clone the repository that contains the Vagrant/Ansible settings:

    git clone https://github.com/CDL-Dryad/vagrant-dryad.git
    cd vagrant-dryad
    
## Preparing for an AWS deployment

You need to make sure that you have credentials for AWS access and ssh access.

You will need to have an access key ID and a secret access key for AWS. This is used to give your local machine permission to create VMs on the AWS infrastructure.

Then, you'll need to create a keypair for yourself. This will be used by vagrant to log in to the VMs that you create, so only you will have access to these VMs unless you explicitly grant access to others. To create the keypair, log in to the aws.amazon.com console, then go to the EC2 dashboard. Click on "Key Pairs" on the left sidebar under "Network and Security." Then create a new key pair for yourself. The private key file `xxx.pem` should automatically download. Save this file somewhere safe on your machine, and note the path.

Now you'll need to set the environment variables that the Vagrantfile needs. In your `.bash_profile` (or wherever you set your environment variables), add the following values:

```
# Amazon credentials
export DRYAD_AWS_ACCESS_KEY_ID=<your access key ID, in single quotes>
export DRYAD_AWS_SECRET_ACCESS_KEY=<your secret access key, in single quotes>
export DRYAD_AWS_KEYPAIR_NAME=<the name of your keypair, in single quotes>
export DRYAD_AWS_PRIVATEKEY_PATH=<the full path to your .pem.txt file, (e.g. ~/.ssh/user.pem.txt) in single quotes>
export DRYAD_AWS_VM_NAME=<the name you want the VM to have in the EC2 console>
```

`DRYAD_AWS_ACCESS_KEY_ID` and `DRYAD_AWS_SECRET_ACCESS_KEY` are the credentials that the AWS machine will use to connect with other AWS services. In most cases, these can be your personal AWS credentials.

Reload your settings when you're done: `source ~/.bash_profile`.

Verify that you have the correct path specified: `more $DRYAD_AWS_PRIVATEKEY_PATH` should give you a cryptic key starting with `-----BEGIN RSA PRIVATE KEY-----`. If not, double-check your path in your .bash_profile and source it again.

## Configuring Dryad

Copy the `ansible/group_vars/all.template` to `ansible/group_vars/all` and set a database password.

When vagrant builds your VM, it uses the values in this file to setup the database.  You must replace the all entries in the file that are surrounded by double hash marks (`##`)
- If you are doing an AWS install
  - `aws.regionName` is the region that the AWS machines will be created in. 

## Building a cloud VM with Amazon AWS

Install the Vagrant-aws plugin: `vagrant plugin install vagrant-aws`

Then download the base box: `vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box`

Verify that you have the correct path specified: `more $DRYAD_AWS_PRIVATEKEY_PATH` should give you a cryptic key starting with `-----BEGIN RSA PRIVATE KEY-----`. If not, double-check your path in your .bash_profile and source it again.

Now run `vagrant up` to create a vagrant VM at Amazon. You should be able to find the public IP and public DNS settings for your instance in the EC2 dashboard: find your instance by clicking on Instances in the left sidebar and selecting your instance.

*DO NOT FORGET TO HALT YOUR MACHINE WHEN YOU ARE DONE. (`vagrant halt`)*

## Accessing the Virtual Machine

After the machine has been created/provisioned successfully, you can log in with

    vagrant ssh
    
Within the virtual machine, the __ubuntu__ user owns the code and installed directory.

On first login, a script will finish installing and initializing Rails and the Dryad system. To run the rails server, use the script `rails_start.sh`.

To shut down the virtual machine, use

    vagrant halt

If you wish to destroy the virtual machine

    vagrant destroy

## Customizing the Vagrant-built VM

Beyond the above required changes, you can further customize the development environment. If you wish to customize further, it's a good idea to familiarize yourself with Vagrant's [command-line interface](http://docs.vagrantup.com/v2/cli/).

## Upgrading your VM

As improvements are added to the vagrant/ansible configurations, you can incorporate the changes by merging in the latest changes from this repo, and running `vagrant provision`. Provisioning will bring your VM up to date without removing existing data or files. You may have to add new values to your `ansible/group_vars/all` file if they are required by newer versions of the ansible playbook, but the Vagrantfile will usually check for these important changes.

## Hacking on this repo

If you change the Vagrantfile, issue the `vagrant reload` command. Vagrant will re-read the configuration and restart your VM. It will not rebuild it from scratch. This is useful for changing ports or adding shared directories

If you change the ansible playbooks or variables, issue the `vagrant provision` command. Vagrant will run ansible and pick up your changes.

In either case, it's best to make sure the change works from start to finish. You can do this by cloning the repo and issuing `vagrant up` again. 

