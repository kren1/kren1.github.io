---
layout: post
title:  "VirtualBox to Azure"
date:   2017-04-18 16:23:45 +0100
categories: jekyll update
---

I have been developing one of my current projects in a VM powered by  VirtualBox. It has been a great experience. I can do all my work in the terminal so I have it setup so that I SSH into my VM. Along with tmux this gives me great freedom. I can have the VM running on my desktop, develop there, then change my mind, go to the living room with my laptop and SSH from there, continuing from there. Along with having the safety of keeping multiple copies of my VMs it feels like my own little cloud.

A couple days ago I realized that my project could easily work in a distributed fashion mostly due to leveraging [GNU parallel](https://www.gnu.org/software/parallel/). I also happened to have access to some Azure credits so I thought I would give it a go. I naively thought that because I have everything nicely packaged in VirtualBox it would be as simple as uploading a file. It turns out there is a little more to it and I would have been better of just redoing my setup on one of the provided Azure images.

## Outline

Broadly speaking I would divide moving from VirtualBox VM to an Azure VM in 3 steps:

1. Prepare the guest OS for Azure
1. Uploading it to Azure
1. Start a VM

### Preparing guest OS

Preparing the guest OS is fairly straightforward, following the [official guide for ubuntu ](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-ubuntu?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) should be sufficient.

The only pitfall I encountered was trying to do it on a 32bit system. Azure doesn't support 32bit VMs, but is not great at complaining about it at this stage. You can follow all the steps in the above guide except `sudo apt-get install walinuxagent`. It complains that it can't find a suitable package although it seems as it should. The reason is quite obvious once you realize it as that package is only available in 64bit version. So don't try this for 32bit systems.

### Uploading image to Azure

This is perhaps the most annoying and time consuming part of this exercise. There are several challenges on the way. Firstly the VM images are quite large and typical home broadband upload speeds are quite poor, so this will take several hours.

With that out of the way let's move on to more interesting stuff. Firstly if you are using VirtualBox the disk is probably in `.vdi` format however Azure wants *fixed size* `.vhd` disk format. Lucky we can easily convert between the two with VBoxManage.  Running `VBoxManage clonehd path/to/image.vdi path/to/target/image.vhd --format VHD` should do the trick.  Unfortunately this is not the end of story as this creates a dynamically resizing `VHD` file and Azure only works with fixed sized ones. This can be done by Hyper-V, by following [this guide](https://blogs.msdn.microsoft.com/virtual_pc_guy/2011/12/28/converting-to-a-fixed-virtual-hard-diskthe-easy-way/) . Using the `Inspect disk` function of Hyper-V it can be verified that the disk is indeed fixed size. With that in hand we can move to actually uploading it.

Broadly speaking I found 4 ways of uploading the image:

* Working through the web interface
* Using [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) (examples use `az` command)
* Using [Azure CLI 1.0](https://docs.microsoft.com/en-us/azure/cli-install-nodejs) (examples use `azure` command)
* Using PowerShell (included for completeness)

The resources I found on the internet about this process don't distinct between them clearly so this is an attempt to clear up the confusion.

What I consider the [main official guide](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/classic/create-upload-vhd) for uploading VMs using Azure CLI 1.0 . There is an 2.0 version of it [here](https://docs.microsoft.com/en-gb/azure/virtual-machines/linux/upload-vhd).

However these guides start from scratch and I don't think are the best way of doing it. In my opinion the best way to start is through the web interface. Before diving deeper it is worth noting that Azure works in two modes *classic* and the new one (without modifier or sometimes called *resource group model*). *classic* mode can't be used with CLI 2.0 and it seems on the way out so I won't discuss it further.

As you might have noticed by looking at the two guides above there are some things that need to be created before we can upload the image. Namely *resource group*, *storage account* and *storage container*. Instead of following those steps you can just spin up a simple vanila instance (from the marketplace) and in doing so create all those things. Then you can just click through the interface to find the storage container where the image of the vanila instance is stored and use that. There you will also see an upload button in the menu from where you can upload your image directly from the browser.  Alternatively running `az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd` should achive the same result. the `key1` can be found in the web interface as one of the *access keys* of the storage account.

### Start the VM

Once you have the blob with your image in the cloud all the hard work is done. All that is left is creating an Image from the blob. Once you've successfully created the image you can create the VM from it and hope for the best!
