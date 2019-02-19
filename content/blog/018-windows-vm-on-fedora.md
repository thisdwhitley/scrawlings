---
title: "Windows VM on Fedora"
subtitle: "...because Linux isn't always a supported OS"
date: 2019-02-15T19:30:32Z
type: "post"
tags: ["libvirt","fedora","windows"]
categories: ["tech"]
showDisclosure: false
draft: false
---

I use Linux every day.  And I have for quite a while now.  But sometimes, large
companies do not exactly cater to people in this demographic...so sometimes I 
need to spin up a Windows VM on my Linux system.  Here is what I've found to
work for me.<!--more-->

If you do not care to read (or about) all the words and explanations, feel free
to skip to the [SHORT VERSION](#tldr)

---

### BACKGROUND

Microsoft® provides
[free Windows virtual machines](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)
to allow customers to test their web browsers, which include Microsoft Edge and
Internet Explorer versions 8 to 11. This VM can be used unrestricted for 90
days.

Because I use Linux every day, I already have a built in hypervisor,
[KVM](https://www.redhat.com/en/topics/virtualization/what-is-KVM).
Unfortunately, that isn't one of the platforms for which Microsoft provides a
VM...

***I have created an Ansible role to accomplish all of this!  Please see
[ansible-role-winVMbuilder](https://github.com/thisdwhitley/ansible-role-winVMbuilder)***

---

### CHALLENGE

I prefer to use the native tool whenever possible and therefore see installing
VirtualBox or Vagrant as tedium.  That means I have to do some work to make it
work...I have a tendency of making things hard on myself, but here are the steps
I followed in order to use Microsoft's free and legal (*I think*) VM's on my
laptop running Fedora.

#### 01 install required packages

There are some utilities that are required and some that make this a lot easier.
In order to install the `virtio-win` package (for drivers) we need to enable its
repository.  So we'll do that and install the necessary packages:

~~~bash
sudo wget https://fedorapeople.org/groups/virt/virtio-win/virtio-win.repo \
  -O /etc/yum.repos.d/virtio-win.repo --no-clobber
sudo dnf install -y unzip virt-manager virt-v2v virtio-win
~~~

#### 02 download the VM image

I will be using a few names over and over throughout this exercise so I go ahead
and assign them to variables.  Be patient as it downloads, it could take a while
depending on the image you choose:

~~~bash
URL=https://az792536.vo.msecnd.net/vms/VMBuild_20180425/VirtualBox/MSEdge/MSEdge.Win10.VirtualBox.zip
ZIP_FILE=$(basename $URL)
VM_NAME=${ZIP%.VirtualBox.zip}
wget -P /tmp $URL
~~~

#### 03 unzip the VM image

The zipped file is not really all that compressed, but it contains an
[OVA](https://fileinfo.com/extension/ova) which is really just an archive file.
Luckily we have tools to convert this file type but first we must unzip the file
we downloaded:

~~~bash
unzip -d /tmp /tmp/$ZIP_FILE
OVA_FILE=$(ls -t /tmp/*.ova | head -n1)
~~~

**NOTE:** *this is sort of a cheezy way to get the `OVA_FILE` because I'm really
just getting the* ***last*** *`ova` file in `/tmp` but I'm going with it*

#### 04 convert to qcow2

I prefer the `qcow2` file format for my VM disk images.  Luckily, the `virt-v2v`
package we installed previously provides a great tool to convert our OVA file to
this format:

~~~bash
virt-v2v -i ova "$OVA" \
         -oc qemu:///system \
         -o libvirt \
         -of qcow2 \
         -os default \
         -n default \
         -on $VM_NAME
~~~

You should really review the entire man page for
[virt-v2v](http://libguestfs.org/virt-v2v.1.html) to get a grasp of just how
powerful it is.  But just to touch on a few items of note in the command above,
we have specified the `default` storage pool and the `default` network for our
new VM.  I chose these because they are most likely pretty universal.

**NOTE:** I experienced some pretty odd behavior with this command.  Sometimes
it failed and other times it did not.  In fact, sometimes it failed, I made *NO*
change, but tried it again and it worked (see [below]({{< ref "#failure" >}}))

#### 05 modify the XML for my network

OK, I just wasn't able to get the network right when I was doing the conversion.
I'm sure it is possible, but I decided to modify the VM via its XML file.  All
virtual machines, regardless of what hypervisor is used, are defined in some
associated file.  So we'll capture the XML of this new one, and then modify it
to the specifications we need:

~~~bash
virsh -c qemu:///system dumpxml $VM_NAME > /tmp/${VM_NAME}.new
sed -e 's/bridge/network/g' \
    -e 's/NAT/default/g' \
    -e "s/type='virtio'/type='e1000'/g" \
    --in-place /tmp/${VM_NAME}.new
~~~

#### 06 recreate the VM with new settings

In order to use the modifications we made, we will `undefine` the VM and then
`create` it again:

~~~bash
virsh -c qemu:///system undefine $VM_NAME
virsh -c qemu:///system create /tmp/${VM_NAME}.new
~~~

Viola!

---

### NEXT STEPS

* The system will boot up and install/update some drivers so give that some time
  and then the VM will need to be rebooted (you can accomplish this from the
  command line with `virsh -c qemu:///system reboot $VM_NAME`)
* <s>Microsoft suggests creating a snapshot of the VM so you can keep going to a
  non-expired VM...but I am going to automate this process which will make that
  step unnecessary</s> I have created an Ansible role to accomplish all of this!
  Please see
[ansible-role-winVMbuilder](https://github.com/thisdwhitley/ansible-role-winVMbuilder)
* I need to modify this VM to do what I need and potentially set it up to
  use a VPN

---

### SIDE NOTES

* I have specified the URI to use in each command related to libvirt (`-c 
  qemu:///system`) because, even if my user is in the libvirt group on a system
  the default URI for a user is `qemu:///session` which is different than root's
  and I wanted to make sure I was consistent in order to avoid confusion...ok,
  the truth is that I didn't avoid that confusion initially...

---

### WHAT I LEARNED

* Sometimes Microsoft isn't all that bad
* I'm glad these tools exist

---

### REFERENCE

<div id="tldr"></div>

* <https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/>
* <https://www.redhat.com/en/blog/importing-vms-kvm-virt-v2v>
* <https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/>

---

### SHORT VERSION

To avoid **tl;dr** enjoy this instead:

~~~bash
#### 01 install required packages
sudo wget https://fedorapeople.org/groups/virt/virtio-win/virtio-win.repo \
  -O /etc/yum.repos.d/virtio-win.repo --no-clobber
sudo dnf install -y unzip virt-manager virt-v2v virtio-win
#### 02 download the VM image
URL=https://az792536.vo.msecnd.net/vms/VMBuild_20180425/VirtualBox/MSEdge/MSEdge.Win10.VirtualBox.zip
ZIP_FILE=$(basename $URL)
VM_NAME=${ZIP%.VirtualBox.zip}
wget -P /tmp $URL
#### 03 unzip the VM image
unzip -d /tmp /tmp/$ZIP_FILE
OVA_FILE=$(ls -t /tmp/*.ova | head -n1)
#### 04 convert to qcow2
virt-v2v -i ova "$OVA" \
         -oc qemu:///system \
         -o libvirt \
         -of qcow2 \
         -os default \
         -n default \
         -on $VM_NAME
#### 05 modify the XML for my network
virsh -c qemu:///system dumpxml $VM_NAME > /tmp/${VM_NAME}.new
sed -e 's/bridge/network/g' \
    -e 's/NAT/default/g' \
    -e "s/type='virtio'/type='e1000'/g" \
    --in-place /tmp/${VM_NAME}.new
#### 06 recreate the VM with new settings
virsh -c qemu:///system undefine $VM_NAME
virsh -c qemu:///system create /tmp/${VM_NAME}.new
~~~

<div id="failure"></div>

---

### ODD FAILURE

In this output, I made ***NO*** change between the invocations...and it worked
the second time.  This seems like Albert Einstein's definition of insanity
(...or perhaps
[someone else's definition](https://www.businessinsider.com.au/misattributed-quotes-2013-10)):

~~~bash
$ virt-v2v -i ova "$OVA" -oc qemu:///system -o libvirt -of qcow2 -os default -n default -on $VM_NAME
[   0.0] Opening the source -i ova /depot/images/libvirt/IE11 - Win7.ova
virt-v2v: warning: could not parse ovf:Name from OVF document
[   0.0] Creating an overlay to protect the source from being modified
[   0.1] Initializing the target -o libvirt -oc qemu:///system -os default
[   0.1] Opening the overlay
libvirt: XML-RPC error : Failed to connect socket to '/run/user/1000/libvirt/libvirt-sock': No such file or directory
virt-v2v: error: libguestfs error: could not connect to libvirt (URI = 
qemu:///session): Failed to connect socket to 
'/run/user/1000/libvirt/libvirt-sock': No such file or directory [code=38 
int1=2]

If reporting bugs, run virt-v2v with debugging enabled and include the 
complete output:

  virt-v2v -v -x [...]
$ virt-v2v -i ova "$OVA" -oc qemu:///system -o libvirt -of qcow2 -os default -n default -on $VM_NAME
[   0.0] Opening the source -i ova /depot/images/libvirt/IE11 - Win7.ova
virt-v2v: warning: could not parse ovf:Name from OVF document
[   0.0] Creating an overlay to protect the source from being modified
[   0.1] Initializing the target -o libvirt -oc qemu:///system -os default
[   0.2] Opening the overlay
[  11.0] Inspecting the overlay
[  11.9] Checking for sufficient free disk space in the guest
[  11.9] Estimating space required on target for each disk
[  11.9] Converting Windows 7 Enterprise to run on KVM
virt-v2v: warning: /usr/share/virt-tools/pnp_wait.exe is missing.  
Firstboot scripts may conflict with PnP.
virt-v2v: This guest has virtio drivers installed.
[  15.6] Mapping filesystem data to avoid copying unused and blank areas
[  16.2] Closing the overlay
[  16.4] Checking if the guest needs BIOS or UEFI to boot
[  16.4] Assigning disks to buses
[  16.4] Copying disk 1/1 to /depot/images/libvirt/IE11.Win7-sda (qcow2)
    (100.00/100%)
[  83.8] Creating output metadata
Pool default refreshed

Domain IE11.Win7 defined from /tmp/v2vlibvirt4b0915.xml

[  83.9] Finishing off
$ # huh?!
~~~

---

[[return to top of page]](#)
