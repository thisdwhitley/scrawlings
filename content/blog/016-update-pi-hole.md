---
title: "Update Pi-hole"
subtitle: "even after it is WAY out of sync"
date: 2019-02-08T23:30:08Z
type: "post"
tags: ["linux","technology","dietpi","raspberrypi","pi"]
categories: ["tech"]
showDisclosure: false
draft: false
---

Even after 
[freeing up some space]({{< ref "015-make-room-on-dietpi.md" >}}) on my
ancient [Raspberry Pi](https://www.raspberrypi.org), I was unable to update
my [Pi-hole](https://pi-hole.net/) installation.  Here are notes on how I was
able to get past that and install the latest version.<!--more-->

If you do not care to read (or about) all the words and explanations, feel free
to skip to the [SHORT VERSION](#tldr)

---

### BACKGROUND

As I noted previously, I have an ancient
[Raspberry Pi](https://www.raspberrypi.org) which I am using solely as a network
wide ad blocker with [Pi-hole](https://pi-hole.net/).  It works well enough, but
I haven't updated it in a long time.  I naively thought it'd get updated when I
updated [DietPi](https://dietpi.com) but that didn't happen.

---

### CHALLENGE

Come to find out, you *cannot* update via DietPi and you cannot update via the
web interface.  All the instructions I found indicated that you must run
`pihole -up` in order to do the update.  I don't know if it was because I had
fallen so far behind or what, but that did not work for me.  Luckily the error
told me exactly what to do next:

~~~bash
root@DietPi:~# pihole -up

  Error: Core Pi-hole repo is missing from system!
  Please re-run install script from https://pi-hole.net
root@DietPi:~#
~~~

I can't say that I totally understand what repo is missing.  But I researched
more and determined I simply needed to run the install script remotely.  Luckily
it was smart enough to realize it was actually an update (saving all my
settings, which is nice).  ***BUT***, it *also* failed with the following,
unhelpful message:

~~~bash
root@DietPi:~# curl -sSL https://install.pi-hole.net | bash
...<snip>...
  [✓] Check for existing repository in /var/www/html/admin
  [i] Update repo in /var/www/html/admin...
  : Could not update local repository. Contact support.
root@DietPi:~#
~~~

I wasn't going to contact support...

---

### SOLUTION

I realize only in hindsight that the error was more helpful than I thought.  It
is telling me which repo it was trying to update.  I don't fully understand how
Pi-hole works, nor the installer, but I looked in that `/var/www/html/admin`
directory and found that it was pointing to a git repo.  I'm not a pro at using
git, but running `git reset --hard` allowed me to move forward:

~~~bash
root@DietPi:~# cd /var/www/html/admin
root@DietPi:/var/www/html/admin# git remote -v
origin	https://github.com/pi-hole/AdminLTE.git (fetch)
origin	https://github.com/pi-hole/AdminLTE.git (push)
root@DietPi:/var/www/html/admin# git reset --hard
HEAD is now at af8c926c Merge pull request #795 from pi-hole/release/v4.0
root@DietPi:~# cd
root@DietPi:~# curl -sSL https://install.pi-hole.net | bash
...<snip>...

  [i] The install log is located at: /etc/pihole/install.log
Update Complete! 

  Current Pi-hole version is v4.2.1
  Current AdminLTE version is v4.2
  Current FTL version is v4.2.1
root@DietPi:~# 
~~~

---

### NEXT STEPS

* I should probably explore how to better utilize the ad blocker but I'm
  currently just using all default settings.  I know you can import other and
  additional blacklists, but I'm not yet

---

### SIDE NOTES

* I do not log into this thing very often, which is part of the reason this was
  never addressed previously.  That's the point of treating it as an appliance,
  *right?!*
* As I mentioned, I'm not git pro so I don't fully know the effect of resetting
  the head as I did.  I also wonder if I should have tried to `git pull` first?

---

### WHAT I LEARNED

* sometimes working with a black-box appliance is harder than it should be

---

### REFERENCE

<div id="tldr"></div>

* I found a discussion that recommended the reset (along with checking network
  connectivity and `/etc/resolv.conf`) but I have lost the page...sorry author
  of other posts

---

### SHORT VERSION

To avoid **tl;dr** enjoy this instead:

~~~bash
root@DietPi:~# cd /var/www/html/admin
root@DietPi:/var/www/html/admin# git reset --hard
HEAD is now at af8c926c Merge pull request #795 from pi-hole/release/v4.0
root@DietPi:~# cd
root@DietPi:~# curl -sSL https://install.pi-hole.net | bash
~~~

---

[[return to top of page]](#)
