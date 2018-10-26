+++
title = "Windows XP VM on Fedora"
date = "2018-10-08"
tags = ["technology","kvm","virtualization"]
categories = ["tech"]
banner = "img/banners/banner-2.jpg"
draft = true
+++

For reasons that I cannot comprehend, I am need to run a Windows XP VM on a
Fedora system.  Tag along, this should be fun!<!--more-->

If you do not care to read (or about) all the words and explanations, feel free
to skip to the [SHORT VERSION](#short-version)

### BACKGROUND

Well, I was going to attempt to protect the innocent here, but I just **must**
explain why I'm doing this.  *It is for my mother.*  She uses an ancient version
of a CAD software that apparently **MUST** be installed on Windows XP.  Every
time we discuss the absurdity of this, one of us gets pissed at the other for
not seeing their side and the conversation goes no where.

I consistently preach the benefits of opensource and have pointed out that we
can work around this ridiculous requirement if she's just willing to entertain
the possibility of alternatives.  I have even shared with her a few
[opensource alternatives to AutoCAD](https://opensource.com/alternatives/autocad).
But, in full disclosure, I don't know anything about AutoCAD nor the
difficulties I might be introducing just by suggesting a switch.

All the same, I figure if I can get Windows XP installed on a Fedora system,
then she's part of the way there.  It might be an evil plan, I haven't decided
yet.

I currently used Fedora as my daily driver, so I am using it for this little
exercise and once I nail down the steps, I'll do them again on a system for her.
I will probably have to modify this post at that point because I already have my
own system set up for virtualization so I'm starting a little be ahead.  All the
same, here are the steps I took:

#### 01 Download Windows XP

Because it is so old and crappy, you can download Windows XP for absolutely
free.  I had to browse around a bit to find the URL I wanted, but once I found
it, just `wget` it from your Linux system:

~~~bash
cd /depot/images/libvirt/
wget https://download.microsoft.com/download/7/2/C/72C7BAB7-2F32-4530-878A-292C20E1845A/WindowsXPMode_en-us.exe
~~~

***NOTE1:*** be aware of the word wrap in that code...I've got to find a theme
that scrolls instead of wraps  
***NOTE2:*** it's almost 500G so give it some time  
***NOTE3:*** I've got my images in `/depot/images/libvirt` but modify according
to your situation  
***NOTE4:*** I sort of think that the long string in that URL might be session
specific so I don't know if that will work in the long term...  

#### 02 Install required utility

What we just downloaded has a `.exe` extension, but it actually contains an
embedded "cabinet" file which is what we want.  We can extract that with the
`cabextract` utility in Fedora, but we must install it first.  I am not a fan of
installing stuff for a single use so I will probably explore alternatives to
this, but for brevity, let's install that:

~~~bash
sudo dnf install -y cabextract
~~~

***NOTE:*** if there is something to note

#### 03 Step3

Description of what the following code will do:

~~~bash
#some
#code
#in here
~~~

---

### NEXT STEPS

Put next steps here

---

### WHAT I LEARNED

* Linux is fun!

---

### REFERENCES

* [How to Legally Download Windows XP for Free, Straight From Microsoft](https://www.makeuseof.com/tag/download-windows-xp-for-free-and-legally-straight-from-microsoft-si/)

<hr id="short-version" />

### SHORT VERSION {#short-version}

To avoid **tl;dr** enjoy this instead:

~~~bash
## 01 steps from above
~~~

[[return to top of page]](#background)
