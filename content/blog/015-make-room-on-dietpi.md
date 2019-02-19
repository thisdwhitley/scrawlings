---
title: "Increase disk space on DietPi"
subtitle: "...by reducing swap..."
date: 2019-02-08T23:03:08Z
type: "post"
tags: ["linux","technology","dietpi","raspberrypi","pi"]
categories: ["tech"]
showDisclosure: false
draft: false
---

I have been unable to update my DietPi installation for some time now because of
disk space constraints.  I finally looked into it and here is what I
found.<!--more-->

If you do not care to read (or about) all the words and explanations, feel free
to skip to the [SHORT VERSION](#tldr)

---

### BACKGROUND

For some reasons that I should really re-evaluate, I installed
[DietPi](https://dietpi.com) on my very early
[Raspberry Pi](https://www.raspberrypi.org) (I'll have to search my archives
because I remember researching the version at some point...but suffice it to say
that this thing is very rudimentary).  Well, basically ever since that initial
installation, it has told me there was an update available.

However, every attempt to run `dietpi-update` resulted in an error stating that
there was not enough disk space available for the update (I don't recall the
exact error wording, sorry about that).  This was puzzling to me because I do
not have much running on this system other than [Pi-hole](https://pi-hole.net)
for network-wide ad blocking.  I do not know much about how this service
functions, but I was surprised that **it** might be the reason I don't have any
disk space.

After poking around on the command line I eventually found that, by default, the
DietPi installation had created and was using a ~1.8G swap file.  There are
endless articles on swap recommendations and I'm sure that DietPi adheres to one
such recommendation...but my entire disk is only 4G so you can see why this
doesn't work for my situation.  (*This post is not going to make any commentary
on how large swap should be or get into any details of what swap is because the
internet abounds with this sort of data: see
[here](https://opensource.com/article/18/9/swap-space-linux-systems) for a
start.*)

---

### CHALLENGE

I found some hints online that indicated I should use the nice TUI that is
provided by DietPi to change the size of swap, that's the point after all, isn't
it: to make this whole thing more user-friendly.  Well I couldn't select the
right order of menu items to modify swap like I needed to...***but I have spent
the majority of my career on a command line and know how to modify swap
there!***

---

### SOLUTION

#### 01 ensure that swap isn't being used

In my case, out of the 1.8G allocated for swap, 0B was being used:

~~~bash
root@DietPi:~# swapon
NAME      TYPE  SIZE USED PRIO
/var/swap file 1.8G    0B   -2                # <--- 1.8G but 0B used...
root@DietPi:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            227          37          39          36         150          89
Swap:          1820           0        1820   # <--- 0 used
~~~

#### 02 turn swap "off"

We need to "disable" or turn swap "off" so that nothing attempts to use it:

~~~bash
root@DietPi:~# swapoff -a
~~~

#### 03 free up the space swap was using

There are a lot of ways to configure swap, but the DietPi installation utilizes
a swap **file** so we will delete that file:

~~~bash
root@DietPi:~# grep swap /etc/fstab
/var/swap    none    swap    sw    0   0                  # <--- see /var/swap
root@DietPi:~# ls -l /var/swap
-rw------- 1 root root 1909456896 Aug  2  2018 /var/swap  # <--- note the size
root@DietPi:~# rm -vfr /var/swap 
removed '/var/swap'
~~~

#### 04 create a new swap file

The simplest strategy here is to reuse the file name that DietPi is expecting,
but just use a smaller file.  I decided to use 1G (for no real reason):

~~~bash
root@DietPi:~# fallocate -l 1G /var/swap  # <--- I used 1G, use what feels right
root@DietPi:~# chmod 0600 /var/swap       # <--- set the perms correctly
root@DietPi:~# mkswap /var/swap           # <--- label it as swap
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=ce36cdd7-2c9d-4bfa-bf27-3a291fa91664
~~~

#### 05 turn swap back "on"

Since we reused the filename, everything should just work as expected after we
reduced the size of the swap file, re-enable it:

~~~bash
root@DietPi:~# swapon -a
root@DietPi:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            227          38          36          36         152          88
Swap:          1023           0        1023   # <--- new size
root@DietPi:~# swapon
NAME      TYPE  SIZE USED PRIO
/var/swap file 1024M   0B   -2                # <--- 1024M now
~~~

---

### NEXT STEPS

* hopefully, after reducing the swap file size, there is enough space to execute
  the `dietpi-update` (it was enough for me)...and keep in mind that the update
  *might* take a while if you missed as many updates as I did...

---

### SIDE NOTES

* I do not log into this thing very often, which is part of the reason this was
  never addressed previously.  That's the point of treating it as an appliance,
  *right?!*
* I do not know what effect reducing the swap size will be, I'm sure the default
  size was picked for a reason, but I'm too cheap to use a larger disk
* If you find that swap is actually being used in your case you may need to
  take some different steps, but I really don't think that disabling it
  momentarily and reducing the size will make *that* big of a difference...but
  YMMV

---

### WHAT I LEARNED

* this is still a great use for my old hardware, I just needed to do some things
  that DietPi might not have wanted...

---

### REFERENCE

<div id="tldr"></div>

* N/A

---

### SHORT VERSION

To avoid **tl;dr** enjoy this instead:

~~~bash
#### 01 ensure that swap isn't being used
root@DietPi:~# swapon
NAME      TYPE  SIZE USED PRIO
/var/swap file 1.8G    0B   -2                # <--- 1.8G but 0B used...
root@DietPi:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            227          37          39          36         150          89
Swap:          1820           0        1820   # <--- 0 used
#### 02 turn swap "off"
root@DietPi:~# swapoff -a
#### 03 free up the space swap was using
root@DietPi:~# grep swap /etc/fstab
/var/swap    none    swap    sw    0   0                  # <--- see /var/swap
root@DietPi:~# ls -l /var/swap
-rw------- 1 root root 1909456896 Aug  2  2018 /var/swap  # <--- note the size
root@DietPi:~# rm -vfr /var/swap 
removed '/var/swap'
#### 04 create a new swap file
root@DietPi:~# fallocate -l 1G /var/swap  # <--- I used 1G, use what feels right
root@DietPi:~# chmod 0600 /var/swap       # <--- set the perms correctly
root@DietPi:~# mkswap /var/swap           # <--- label it as swap
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=ce36cdd7-2c9d-4bfa-bf27-3a291fa91664
#### 05 turn swap back "on"
root@DietPi:~# swapon -a
root@DietPi:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            227          38          36          36         152          88
Swap:          1023           0        1023   # <--- new size
root@DietPi:~# swapon
NAME      TYPE  SIZE USED PRIO
/var/swap file 1024M   0B   -2                # <--- 1024M now
~~~

---

[[return to top of page]](#)
