---
date: 2019-07-09
layout: post
title: "Setup a Coreboot Buildroot with Vagrant"
description: How to get up an running with a coreboot buildroot in a breeze.
categories: [coreboot]
tags: [coreboot, privacy, security, trusted computing, bios, coreboot buildroot, vagrant, firmware, libreboot, opensource, intel me, intel management engine]
---

Ive struggled a bit arround getting the buildroot to sucessfully compile on my linux distribution because my gcc version is a little bit newer then the one that gets installed by default on an ubuntu system. So there are two options. You start hacking arround your main os and downgrade what not to get the buildroot running and hope that it will workout somehow or you simply setup yourself a Ubuntu vm with all the needed dependencies.

With this short Vagrant file you should be up and running in a breeze without having any headaches.

Create a Folder and put this `Vagrantfile` into it.

{% highlight ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  config.vm.box = "ubuntu/bionic64"

  config.vm.provider "virtualbox" do |vb|
     vb.gui = true
  
     vb.memory = "8192"
  end
  config.vm.provision "shell", inline: <<-SHELL

    sudo apt-get update
    sudo DEBIAN_FRONTEND=noninteractive apt-get -y upgrade
    
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -y python bison build-essential curl flex git gnat libncurses5-dev m4 zlib1g-dev
    git clone https://review.coreboot.org/coreboot ~/coreboot
  SHELL
end
{% endhighlight %}

After running the command `vagrant up` in the folder a VirtualBox Ubuntu Bionic 64 VM should get created and you will be able to access your buildroot by sshing to the virtual machine by using the command `vagrant ssh`.

Happy hacking!

09.07.2019 Update.

The article just got updated to move the source away from the shared folder, since i found out the hard way that it will give you a lot of compile errors otherwise.
