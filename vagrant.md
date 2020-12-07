## Table of contents

- [Table of contents](#table-of-contents)
- [General commands](#general-commands)
- [Vagrantfile](#vagrantfile)
  - [Provisioners](#provisioners)
  - [Networking](#networking)
  - [Multiple VMs](#multiple-vms)
- [Boxes](#boxes)
- [VirtualBox Guest tools issues](#virtualbox-guest-tools-issues)

## General commands

```sh
vagrant init $boxname            # creates the Vagrantfile
vagrant up [$vm_name]            # runs the VM; note that a Vagrantfile is internally associated with a path, so moving it will cause vagrant not to
                                 # $vm_name is used when multiple machines are defined.
vagrant ssh                      # connects via ssh to a machine

vagrant (suspend|resume)
vagrant halt                     # graceful shutdown
vagrant destroy                  # next up command will need the env to be rebuilt
```

## Vagrantfile

Sample:

```ruby
Vagrant::Config.run do |config|
  config.vm.box = "test"

  # prepare the hostname automatically
  config.vm.host_name = "vmsav-dbs"

  # Port forwarding
  config.vm.forward_port 80, 4567

  # Memory
  config.vm.customize ["modifyvm", :id, "--memory", 3072]
end
```

Load order:

- gem directory
- box
- home
- project

### Provisioners

Chef solo:

```ruby
# Enable and configure the chef solo provisioner
# When a provisioner is added, at least one recipe must be specified.
config.vm.provision :chef_solo do |chef|
  # Download a cookbook
  chef.recipe_url = "http://files.vagrantup.com/getting_started/cookbooks.tar.gz"

  # Run the recipe
  chef.add_recipe("vagrant_main")
end
```

Chef server:

```ruby
config.vm.provision :chef_client do |chef|
  chef.chef_server_url = "http://mychefserver.com:4000"

  # Absolute path, or relative to project directory
  chef.validation_key_path = "validation.pem"

  # Provision with the apache2 recipe
  chef.add_recipe("apache2")

  # Provision with the database role
  chef.add_role("database")

  # Modifying the run list directly
  chef.run_list = ["recipe[foo]", "role[bar]"]

  # Set the environment for the chef server
  chef.environment = "development"

  # Other advanced options
  chef.validation_client_name = "chef-validator"
  chef.client_key_path = "/etc/chef/client.pem"

  # Important!! If not deleted, the nodes will prevent assignment of future clients ("Client already exists")
  # The underlying naming is `vagrant-ubuntu-oneiric`.
  chef.delete_node = true
  chef.delete_client = true
end
```

Shell provisioner:

```ruby
config.vm.provision :shell, path: "test.sh"
config.vm.provision :shell, inline: "echo foo > /vagrant/test"

# Pass params. WATCH OUT!: params are not automatically escaped
config.vm.provision :shell do |shell|
  shell.inline = "echo $1 > /vagrant/test"
  shell.args = "'write this to a file'"
end
```

### Networking

```ruby
# Host-only
# netmask is 255.255.255.0 by default
config.vm.network :hostonly, "10.11.12.13"[, netmask: "255.255.0.0"]

# Bridged. The default adapter used is #3
config.vm.network :bridged
```

### Multiple VMs

In multiple VMs case, `vagrant up` will run all the machines, unless the `$vm_name` is specified.

```ruby
config.vm.define :web do |web_config|
  web_config.vm.box = "web"
  web_config.vm.forward_port 80, 8080
end

config.vm.define :db do |db_config|
  db_config.vm.box = "db"
  db_config.vm.forward_port 3306, 3306
end
```

## Boxes

Create a box from a virtual machine.

```sh
# Disks are automatically converted to VMDK.
# Machine is shutdown if running
#
vagrant package [--vagrantfile /path/to/Vagrantfile] [--include $otherfile] [--output $boxname.box] $vm_name
```


```sh
vagrant box add --name $box_name [--force] $box_file_name # Import a box; [force] overwrite an existing box
vagrant box list
vagrant box remove $boxname
```

## VirtualBox Guest tools issues

Vagrant: fix mismatching guest tools version.  
Reload the machine after this; use  `VBoxManage list vms` for gathering UUIDs.

```sh
VBoxManage guestproperty set $machine_uuid /VirtualBox/GuestAdd/Version
```

Note that his may not work; if for example, the host reports `5.0.16 r105871` and the guest reports `5.0.16`, then check that the guest has the right version, then manually set it:

```sh
VBoxManage guestproperty set <machine_uuid> /VirtualBox/GuestAdd/Version <short_version>
```
