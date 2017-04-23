---
layout: post
title:  "VirtualBox to Azure"
date:   2017-04-18 16:23:45 +0100
categories: jekyll update
---

Developing one of my recent projects in a VirtualBox VM powered has been a great experience. I have no need for GUI as all my work is in the terminal. I usually SSH into my VM for a better experience (mostly copying stuff). It also  enables me to move around while working. I can have the VM running on my desktop, develop there, then change my mind, go to the living room on my laptop and continue where I left of (thanks to tmux).

A couple days ago I realized that my project could easily work in a distributed fashion mostly due to leveraging [GNU parallel](https://www.gnu.org/software/parallel/). I also happened to have access to some Azure credits so I thought I would give it a go. I naively thought that because I have everything nicely setup in VirtualBox it would be as simple as uploading a file. It turns out there is a little more to it and I would have been better off just redoing my setup on one of the provided Azure images.

## Outline

Broadly speaking I would divide moving from VirtualBox VM to an Azure VM in 3 steps:

1. Prepare the guest OS for Azure
1. Uploading it to Azure
1. Start a VM

### Preparing guest OS

Preparing the guest OS is fairly straightforward, following the [official guide for ubuntu ](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-ubuntu?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) should be sufficient.

The only pitfall I encountered was trying to do it on a 32bit system. Azure doesn't support 32bit VMs, but is not great at complaining about it. You can follow all the steps in the guide except `sudo apt-get install walinuxagent`. It complains that it can't find a suitable package although it seems as it should. The reason is quite obvious once you realize it as that package is only available for 64 bit systems.

### Uploading image to Azure

This is perhaps the most annoying and time consuming part of this exercise. The VM images are quite large and typical home broadband upload speeds are quite poor, so this will take several hours.

With that out of the way let's move on to more interesting problems. Firstly if you are using VirtualBox, the disk is probably in a `.vdi` format. Azure however wants it in *fixed size* `.vhd` disk format. At first glance disk images formats can easily be converted between eachother with VBoxManage.  Running `VBoxManage clonehd path/to/image.vdi path/to/target/image.vhd --format VHD` should do the trick for this particular case.  Unfortunately this is not quite right as ir creates a dynamically resizing `VHD` file and Azure only works with fixed sized ones. You can "fix" the size with Hyper-V, by following [this guide](https://blogs.msdn.microsoft.com/virtual_pc_guy/2011/12/28/converting-to-a-fixed-virtual-hard-diskthe-easy-way/) . The `Inspect disk` function of Hyper-V can be used to verify that the disk is indeed fixed size. With that in hand we can move to actually uploading it.

Broadly speaking I found 4 ways of uploading the image:

* Working through the web interface
* Using [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) (examples use `az` command)
* Using [Azure CLI 1.0](https://docs.microsoft.com/en-us/azure/cli-install-nodejs) (examples use `azure` command)
* Using PowerShell (included for completeness)

The resources I found on the internet about this process don't distinct between them clearly so this is an attempt to clear up the confusion.

The CLI 1.0 version of the whole upload guide is [this](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/classic/create-upload-vhd) with the 2.0 version being [this](https://docs.microsoft.com/en-gb/azure/virtual-machines/linux/upload-vhd).

It is worth noting that Azure works in two modes *classic* and the new one (without modifier or sometimes called *resource group model*). *classic* mode can't be used with CLI 2.0. However the CLI 1.0 version of the upload guide uses the *classic* model to upload the images.

The two guides have a lot of boilerplate in creating  *resource group*, *storage account* and *storage container*, which you really shouldn't concern yourself much about at this point. Instead of following those steps you can just spin up a simple vanila instance (from the marketplace) and in doing so create all those things. Then you can just click through the interface to find the storage container where the image of the vanila instance is stored and use that. There you will also see an upload button in the menu from where you can upload your image directly from the browser (note that the upload can be paused or interrupted and then resumed).  Alternatively running `az storage blob upload --account-name mystorageaccount \
    --account-key key1 --container-name mydisks --type page \
    --file /path/to/disk/mydisk.vhd --name myDisk.vhd` should achieve the same result. the `key1` can be found in the web interface as one of the *access keys* of the storage account as seen in the image.

![location of the key1](/assets/azure-key1.png)

### Start the VM

Once you have successfully uploaded the disk image as a blob, I see two ways of procceding. The first (and probably the right one) is to create an azure image out of it and spawn VMs out of that. This way you can control things like SSH keys from the dashboard. However it comes at a cost of deleting all non-root users, which was not appropriate for me. It is also more likely that your VM will fail to boot if you go with this approach.

The second way is to create a disk out of the image and then attach that to a VM, which is what I've done, this is also the way described in the CLI 2.0 version of the [upload guide](https://docs.microsoft.com/en-gb/azure/virtual-machines/linux/upload-vhd).
