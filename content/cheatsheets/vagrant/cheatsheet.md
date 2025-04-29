---
title: Vagrant CheatSheet
linktitle: Vagrant CheatSheet
type: book
date: "2019-05-05T00:00:00+01:00"
toc: true
tags:
  - Vagrant

### Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

<!--more-->

### Terms and Definitions
- `Box`: A prepackaged Vagrant environment that is the same on any computer in which it is provisioned.
- `Provider`: The underlying system that manages the virtual machines, such as VirtualBox and Docker.
- `Provisioner`: Systems that allow you to install software and otherwise configure guest machines, such as Chef and Puppet.
- `Vagrantfile`: A single file used to define the desired Vagrant environment; written in Ruby.

Typing `vagrant` from the command line will display a list of all available commands.

Be sure that you are in the same directory as the Vagrantfile when running these commands!

### Creating a VM
- `vagrant init`           -- Initialize Vagrant with a Vagrantfile and ./.vagrant directory, using no specified base image. Before you can do vagrant up, you'll need to specify a base image in the Vagrantfile.
- `vagrant init <boxpath>` -- Initialize Vagrant with a specific box. To find a box, go to the [Vagrant Cloud](https://app.vagrantup.com/boxes/search). When you find one you like, just replace it's name with boxpath. For example, `vagrant init ubuntu/trusty64`.

### Starting a VM
- `vagrant up`                  -- starts vagrant environment (also provisions only on the FIRST vagrant up)
- `vagrant resume`              -- resume a suspended machine (vagrant up works just fine for this as well)
- `vagrant provision`           -- forces reprovisioning of the vagrant machine
- `vagrant reload`              -- restarts vagrant machine, loads new Vagrantfile configuration
- `vagrant reload --provision`  -- restart the virtual machine and force provisioning

### Getting into a VM
- `vagrant ssh`           -- connects to machine via SSH
- `vagrant ssh <boxname>` -- If you give your box a name in your Vagrantfile, you can ssh into it with boxname. Works from any directory.

### Stopping a VM
- `vagrant halt`        -- stops the vagrant machine
- `vagrant suspend`     -- suspends a virtual machine (remembers state)

### Cleaning Up a VM
- `vagrant destroy`     -- stops and deletes all traces of the vagrant machine
- `vagrant destroy -f`   -- same as above, without confirmation
- `vagrant box list | cut -f 1 -d ' ' | xargs -L 1 vagrant box remove -f` -- remove ALL your local Vagrant Boxes

{{% callout note %}}

Assuming you have only one box per provider, this command will delete ALL Vagrant boxes you currently have on your system. This command does the following:

- vagrant box list: Prints out a list of all installed vagrant boxes (with two columns—box name or path, and meta info)
- cut -f 1 -d ' ': Cuts the list and takes out just the first column (using spaces to delimit the columns)
- xargs -L 1 vagrant box remove -f: Use xargs to run one command per line, running the command vagrant box remove -f [box name from list/cut].

You can use xargs' -t option to output the commands being run just before they're executed. And if you have multiple boxes per provider, or if you have multiple versions of the same box, you'll likely need to modify the command a bit like so:
```
vagrant box list | cut -f 1 -d ' ' | xargs -L 1 vagrant box remove -f --all
```
{{% /callout %}}

### Boxes
- `vagrant box list`              -- see a list of all installed boxes on your computer
- `vagrant box add <name> <url>`  -- download a box image to your computer
- `vagrant box outdated`          -- check for updates vagrant box update
- `vagrant boxes remove <name>`   -- deletes a box from the machine
- `vagrant package`               -- packages a running virtualbox env in a reusable box

### Saving Progress
-`vagrant snapshot save [options] [vm-name] <name>` -- vm-name is often `default`. Allows us to save so that we can rollback at a later time

### Tips
- `vagrant -v`                    -- get the vagrant version
- `vagrant status`                -- outputs status of the vagrant machine
- `vagrant global-status`         -- outputs status of all vagrant machines
- `vagrant global-status --prune` -- same as above, but prunes invalid entries
- `vagrant provision --debug`     -- use the debug flag to increase the verbosity of the output
- `vagrant push`                  -- yes, vagrant can be configured to [deploy code](http://docs.vagrantup.com/v2/push/index.html)!
- `vagrant up --provision | tee provision.log`  -- Runs `vagrant up`, forces provisioning and logs all output to a file

### Plugins
- [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater) : `$ vagrant plugin install vagrant-hostsupdater` -- plugin to update your `/etc/hosts` file automatically each time you start/stop your vagrant box.
- [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) : `$ vagrant plugin install vagrant-vbguest` -- plugin which automatically installs the host's VirtualBox Guest Additions on the guest system.

### Notes
- If you are using [VVV](https://github.com/varying-vagrant-vagrants/vvv/), you can enable xdebug by running `vagrant ssh` and then `xdebug_on` from the virtual machine's CLI.
