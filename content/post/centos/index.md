---
title: Installation of CentOS 7.0 with VirtualBox
subtitle:

# Summary for listings and search engines
summary:

# Link this post with a project
projects: []

# Date published
date: "2020-12-13T00:00:00Z"

toc: true

# Date updated
lastmod: "2020-12-13T00:00:00Z"

# Is this an unpublished draft?
draft: false

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

categories:

---

<!--more-->

### Overview

As part of my upcoming HowTo tutorials on my blog, I will need to install CentOS 7 on a virtualbox. My goal would be to assign a static IP for these machines and use them as Docker hosts.

For those who may not know this, ```VirtualBox``` is a 🆓, open source cross platform hypervisor software that allows users to run multiple operating systems from within a single machine. In other words, you can spin up multiple virtual machines of any desired OS within minutes of each other as long as the underlining machine spec can handle the load.

### Get Started

The things that you will need:

- 👉 [Install VirtualBox](https://www.virtualbox.org/wiki/Downloads/)
- 👉 [Download the DVD ISO Image](https://www.centos.org/download/)
- 👉 [Virtualization turned on from BIOS/UEFI settings](https://learn.microsoft.com/en-us/windows/wsl/install-manual#step-3---enable-virtual-machine-feature)


{{% callout note %}}

* I'm working on a Windows 10 machine, so I used the CentOS-7-x86_64-DVD-2009.iso
* Minimum Requirements to Install CentOS 7 on VirtualBox
  * 10GB minimum Storage, 20GB+ recommended.
  * 2GB minimum RAM, 4GB+ recommended.
  * Dual-Core Processor.

{{% /callout %}}

{{% stepper %}}
<div class="step">
  
  ## Step 1:
  Install VirtualBox and open it.
  - First make sure you do have a host network created. Also make a note of the IPv4 address and the Subnet mask. If none exists by default create one here:
  ![Host Network](/images/uploads/Centos-1.PNG)

</div>
<div class="step">

  ## Step2:
  - Select the Linux and then Red Hat, since CentOS is the clone of Red Hat and uses a similar architecture.
  - Set the memory size to 4GB.
  - Select "Create a Virtual Hard Disk"
    ![](/images/uploads/Centos-2.PNG)

</div>
<div class="step">
  
  ## Step3:
  - Set your desired hard disk size and type and create:
  ![](/images/uploads/Centos-3.PNG)

</div>
<div class="step">
  
  ## Step4:

  Now we will change some of the settings for the VM. Some of them maybe optional for all use cases but these were the ones I did.

  ![](/images/uploads/Centos-4.PNG)
  ![](/images/uploads/Centos-5.PNG)
  ![](/images/uploads/Centos-6.PNG)
  ![](/images/uploads/Centos-7.PNG)

  ![](/images/uploads/Centos-8.PNG)
  ![](/images/uploads/virtualbox-nat.png)

  ![](/images/uploads/Centos-9.PNG)
  ![](/images/uploads/virtualbox-hostonly.png)
  
</div>
<div class="step">
  
  ## Step5:

  Now we would start the VMs and it would pick the DVD.iso image and start the installation. The installer has a GUI which would pretty much walk you through the respective steps.
    ![](/images/uploads/Centos-10.PNG)
    ![](/images/uploads/Centos-11.PNG)
    ![](/images/uploads/Centos-12.PNG)
  
</div>
<div class="step">
  
  ## Step6:

  We would need to make sure the machine would have internet connectivity and a static IP. These settings needs to be set at this step. The installer has this weird setting to have the internet adaptors turned off by default. Make sure you turn it on here:

  ![](/images/uploads/Centos-13.PNG)
  ![](/images/uploads/Centos-14.PNG)
  ![](/images/uploads/Centos-15.PNG)
  ![](/images/uploads/Centos-17.PNG)
  ![](/images/uploads/Centos-18.PNG)  
  ![](/images/uploads/Centos-19.PNG)  
  ![](/images/uploads/Centos-20.PNG)  
  ![](/images/uploads/Centos-21.PNG)  
  ![](/images/uploads/Centos-22.PNG)    
  ![](/images/uploads/Centos-23.PNG)
  ![](/images/uploads/Centos-24.PNG)  
  ![](/images/uploads/Centos-25.PNG)
  ![](/images/uploads/Centos-26.PNG)        
  ![](/images/uploads/Centos-27.PNG)  
  ![](/images/uploads/Centos-28.PNG) 
  
</div>
<div class="step">
  
  ## Step7:

  Install Guest Additions: 
  
</div>
{{% /stepper %}}
