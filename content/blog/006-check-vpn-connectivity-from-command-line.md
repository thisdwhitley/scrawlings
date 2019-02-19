---
title: "Check VPN connectivity from command line"
date: "2018-10-11"
type: "post"
tags: ["technology","vpn","command line","bash","linux"]
categories: ["tech"]
showDisclosure: true
---

I pay [TorGuard](https://torguard.net/aff.php?aff=4689) (affiliate link) for VPN
services.  Mainly because sometimes I do some shady shit (that's alliteration).
No matter what I'm doing, it is often worthwhile to see how my web browsing
traffic looks to the outside world.  So here are some notes on how I can check
that from a command line.<!--more-->

If you do not care to read (or about) all the words and explanations, feel free
to skip to the [SHORT VERSION](#short-version)

### BACKGROUND

You might be surprised just how much information can be gleaned from *just* your
IP address.  I won't go into the depths of that discussion, but for fun, open
another browser or tab and visit:
[Whats My IP](https://torguard.net/whats-my-ip.php) (their spelling, not mine...
I'd put an apostrophe before that "s")

That's a lot of information based just on your IP and browser, right?!?!

But enough about ***WHY*** I might use a VPN sometimes.  Let's talk about how we
can check to see if I'm connected to the internet through the VPN.  More
precisely, we want to see where an external site thinks my IP address is
located.

#### 01 From myself

This command will utilize information from [DNS Leak Tests](http://dnsleak.com):

~~~bash
curl -ks https://www.dnsleaktest.com | grep flag | sed -e 's/.*from \(.*\),.*/\1/'
~~~

***NOTE:*** there are a lot of ways to accomplish this from a lot of different
sites, this is just what *I* use

#### 02 From containers

Sometimes I want to see how my containers appear to originate from.  So here is
a command to execute this same command on all my running containers with a
little bit of formatting for readability:

~~~bash
for I in $(docker ps --format '{{.Names}}'); do \
  location=$(docker exec -it $I curl -ks https://www.dnsleaktest.com \
  | grep flag | sed -e 's/.*from \(.*\),.*/\1/'); \
  printf "%-15s%s\n" "$I:" "$location"; \
done
~~~

***NOTE:*** I've formatted this a bit, but it ***is*** a single command that can
be cut-and-pasted on a system that has docker containers running

---

### NEXT STEPS

I've considered creating an Ansible playbook for this that I can schedule and
run from AWX...but that might be overkill.

---

### WHAT I LEARNED

* Linux is fun!

---

### REFERENCES

* [I've hinted at this before](http://www.ilearnedthisthehardway.com/check-vpn-connectivity/)

---

### SHORT VERSION {#short-version}

To avoid **tl;dr** enjoy this instead:

~~~bash
## 01 From myself
curl -ks https://www.dnsleaktest.com | grep flag | sed -e 's/.*from \(.*\),.*/\1/'

## 02 From containers
for I in $(docker ps --format '{{.Names}}'); do \
  location=$(docker exec -it $I curl -ks https://www.dnsleaktest.com \
  | grep flag | sed -e 's/.*from \(.*\),.*/\1/'); \
  printf "%-15s%s\n" "$I:" "$location"; \
done
~~~

[[return to top of page]](#heading-breadcrumbs)
