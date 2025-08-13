---
title: Installation of CentOS 8.0 with Vagrant
subtitle:

# Summary for listings and search engines
summary:

# Link this post with a project
projects: []

# Date published
date: "2021-04-03T00:00:00Z"

# Date updated
lastmod: "2021-04-03T00:00:00Z"

# Is this an unpublished draft?
draft: false

toc: true

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption:
  focal_point: "Center"
  placement: 1
  preview_only: false

authors:
- admin

tags:
- Unix
- CentOS
- VirtualBox
- Vagrant

categories:

---

<!--more-->

### Overview

As part of my upcoming "HowTo" tutorials on my blog, I will need to install different Linux distros like CentOS, Ubuntu... on a Windows machine. My goal would be to assign a static IP for these machines and use them as ```Docker``` hosts.

And one of the best ways to do that is via a tool called [Vagrant](https://developer.hashicorp.com/vagrant), and that's what I'll be showing here. You can find the different Linux distros via [Vagrant Cloud](https://app.vagrantup.com/boxes/search). And we use them to spin up VMs and tear them down once done very quickly.

```Vagrant``` works with different hypervisor providers like ```VirtualBox```, ```Docker```, ```Hyper-V```, ```VMware```. We will be using ```VirtualBox``` here which is a free, open source cross platform hypervisor software.

In a previous article I showed how to [Install of CentOS 7.0 with VirtualBox]({{< ref "/post/centos/index.md" >}} "Centos 7").
In this article I want to script/automate all the clicking around in ```VirtualBox```, check them into version control, and keep a nice history of them. Vagrant is just automating the process of creating a VM inside of VirtualBox.

### Get Started

The things that you will need:

- 👉 [Vagrant](https://www.vagrantup.com/downloads)
- 👉 [VirtualBox](https://www.virtualbox.org/wiki/Downloads/)

Install Vagrant and open a command terminal and make sure vagrant cli is available.
```
$ vagrant --version
Vagrant 2.2.15
```

Similarly make sure VirtualBox is installed and available
```
$ VBoxManage --version
6.1.18r142142
```

We could be creating multiple VMs potentially via vagrant and we would want them to communicate with each other.
In order to do that we would need to connect the VMs to a **host-only** network. Read about different networking options in this other post [VirtualBox Networking](/cheatsheets/virtualbox/networking/).

With the default VirtualBox installation a host-only network should get created automatically.
We would need to verify one exists or else create one like so:

```
$ VBoxManage list hostonlyifs
```
or else create one:
```
$ VBoxManage hostonlyif create
```

From the official centos site get the link to [Vagrant Image](https://www.centos.org/download/). This routes to [Vagrant Cloud](https://app.vagrantup.com/boxes/search) site where you could find other official/otherwise images for different Linux distros.

On the command line follow the instructions as below:

  ```
  $ vagrant init centos/8
  A `Vagrantfile` has been placed in this directory. You are now
  ready to `vagrant up` your first virtual environment! Please read
  the comments in the Vagrantfile as well as documentation on
  `vagrantup.com` for more information on using Vagrant.
  ```

  This will create a Vagrantfile at the same location and all the aspects of VirtualBox configuration could be applied here.  We'll get into it as we need to. This file will be written in ruby and we will walk through the different sections here next.

  ```
  # -*- mode: ruby -*-
  # vi: set ft=ruby :

  # All Vagrant configuration is done below. The "2" in Vagrant.configure
  # configures the configuration version (we support older styles for
  # backwards compatibility). Please don't change it unless you know what
  # you're doing.
  Vagrant.configure("2") do |config|
    # The most common configuration options are documented and commented below.
    # For a complete reference, please see the online documentation at
    # https://docs.vagrantup.com.

    # Every Vagrant development environment requires a box. You can search for
    # boxes at https://vagrantcloud.com/search.
    config.vm.box = "centos/8"

    # Disable automatic box update checking. If you disable this, then
    # boxes will only be checked for updates when the user runs
    # `vagrant box outdated`. This is not recommended.
    # config.vm.box_check_update = false

    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine. In the example below,
    # accessing "localhost:8080" will access port 80 on the guest machine.
    # NOTE: This will enable public access to the opened port
    # config.vm.network "forwarded_port", guest: 80, host: 8080

    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine and only allow access
    # via 127.0.0.1 to disable public access
    # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    # config.vm.network "private_network", ip: "192.168.33.10"

    # Create a public network, which generally matched to bridged network.
    # Bridged networks make the machine appear as another physical device on
    # your network.
    # config.vm.network "public_network"

    # Share an additional folder to the guest VM. The first argument is
    # the path on the host to the actual folder. The second argument is
    # the path on the guest to mount the folder. And the optional third
    # argument is a set of non-required options.
    # config.vm.synced_folder "../data", "/vagrant_data"

    # Provider-specific configuration so you can fine-tune various
    # backing providers for Vagrant. These expose provider-specific options.
    # Example for VirtualBox:
    #
    # config.vm.provider "virtualbox" do |vb|
    #   # Display the VirtualBox GUI when booting the machine
    #   vb.gui = true
    #
    #   # Customize the amount of memory on the VM:
    #   vb.memory = "1024"
    # end
    #
    # View the documentation for the provider you are using for more
    # information on available options.

    # Enable provisioning with a shell script. Additional provisioners such as
    # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
    # documentation for more information about their specific syntax and use.
    # config.vm.provision "shell", inline: <<-SHELL
    #   apt-get update
    #   apt-get install -y apache2
    # SHELL
  end

  ```

The file could look intimidating at first glance but if you look closely it's basically a lot of helpful comments/instructions to get you started. <br>

Next I will break down the **Vagrantfile** instructions below :

### Vagrant File

  * Section 1: Install Required Plugins

    ```
    # All Vagrant configuration is done below. The "2" in Vagrant.configure
    # configures the configuration version (we support older styles for
    # backwards compatibility). Please don't change it unless you know what
    # you're doing.
    Vagrant.configure("2") do |config|

      # require plugin https://github.com/dotless-de/vagrant-vbguest
      # vagrant-vbguest is a Vagrant plugin which automatically installs the host's
      # VirtualBox Guest Additions on the guest system.
      config.vagrant.plugins = "vagrant-vbguest"
    ```

  {{% callout note %}}
  The official centos box images we are using **centos/8** do not come preconfigured with GuestAdditions. vagrant-vbguest is a Vagrant plugin which automatically installs the host's VirtualBox Guest Additions on the guest system.
  {{% /callout %}}

  * Section 2: Operating System:

    ```
      config.vm.box = VAGRANT_BOX
      # Actual machine name
      config.vm.hostname = VM_NAME
      # :allow_kernel_upgrade (default: false): If true, instead of trying to find matching
      # the matching kernel-devel package to the installed kernel version, the kernel will
      # be updated and the (now matching) up-to-date kernel-devel will be installed.
      # NOTE: This will trigger a reboot of the box.
      config.vbguest.installer_options = { allow_kernel_upgrade: true }
    ```

  * Section 3: Provider Settings:

    ```
    # Set VM name in Virtualbox
      config.vm.provider "virtualbox" do |v|
        v.name = VM_NAME
        v.memory = 2048
      end
    ```
  * Section 4: Network Settings:
    ```
     config.vm.network "private_network", ip: "192.168.56.30", :name => 'VirtualBox Host-Only Ethernet Adapter'
    ```

  {{% callout note %}}
  Vagrant assumes there is an available NAT device on eth0. This ensures that Vagrant always has a way of communicating with the guest machine. In VirtualBox, this assumption means that network adapter 1 is a NAT device.
  {{% /callout %}}

  * Section 5: Folder Settings:
    Sync a folder on the host machine to the guest machine
    ```
     config.vm.synced_folder HOST_PATH, GUEST_PATH
    ```
  * Section 6: Provision Settings:
    Here we could run any provisioning scripts (vagrant provides several provisioners.                             Check them out [here](https://www.vagrantup.com/docs/provisioning))
    ```
    config.vm.provision "shell",inline: $install_docker_script, privileged: true

    $install_docker_script = <<SCRIPT
    echo "Installing dependencies ..."
    sudo apt-get update
    echo Installing Docker...
    curl -sSL https://get.docker.com/ | sh
    sudo usermod -aG docker vagrant
    SCRIPT
    ```

Next we will do **vagrant up**. vagrant up means create the VM and start it up.
And also in that case, vagrant will pull down the box image from vagrant cloud if we don't have it already.

  ```
  $ vagrant up
  ```

That's It! We have a Centos 8 machine running, ready to be SSH'ed into. You can verify by checking your VirtualBox page:
![](/images/uploads/virtualbox.PNG).

### Next Steps
Here's some of the other useful vagrant commands to control the VM lifecycle:

* To SSH into the box type:
  ```
  $ vagrant ssh
  [vagrant@localhost ~]$
  ```

* To turn your VM on, navigate to the directory with your Vagrantfile:
  ```
  vagrant up
  ```
* To pause your VM, navigate to the directory with your Vagrantfile:
  ```
  vagrant suspend
  ```
* To turn your VM off, navigate to the directory with your Vagrantfile:
  ```
  vagrant halt
  ```
* To destroy your VM, navigate to the directory with your Vagrantfile:
  ```
  vagrant destroy
  ```
* Print out a list of all installed vagrant boxes (with two columns—box name or path, and meta info)
  ```
  vagrant box list
  ```

These are just a few of the immediate basic commands you’ll want to learn while using Vagrant. I have got a [CheatSheet](/cheatsheets/vagrant) for your quick reference. For a much more in-depth manual, check out the [docs](https://www.vagrantup.com/docs/index) from Vagrant.
