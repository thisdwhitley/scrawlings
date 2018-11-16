---
title: Blog notes
subtitle: and thought
date: 2018-10-25
tags: ["technology","netlify","hugo","beautifulhugo"]
categories: ["tech"]
draft: true
---

I would like to make a note on my blog.  Currently I'm using hugo, github, and
netlify.  I'm ***currently*** using the
[beautifulhugo](https://github.com/halogenica/beautifulhugo) theme, but here are
some things that I'd like to change (I need to learn more about
[submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)):

- <s>change the colors to red</s> (more notes here)
- make note that links may be affiliate
- have categories in the top bar

And there is a fork that has some items that I like and need to reproduce:

- find in the top bar
- small "share" icons
- similar posts (based on tags)
- categories
- docker social page

<https://github.com/pad92/beautifulhugo> and here it is in action:
[Pad's Notes](https://notes.depad.fr)

## disclosure/disclaimer

I really like the brief disclosure here:
<https://www.runthemoney.com/finances-killing-quality-of-life/>
And it links to his great lengthier stuff here:
<https://www.runthemoney.com/disclosure-policy-disclaimer/>

## submodules

I can't seem to get this [submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
bullshit to work.  So I have followed the instructions
[here](https://gist.github.com/myusuf3/7f645819ded92bda6677) in order to remove the submodule:

1. `echo "" > .gitmodules`
2. `git add .gitmodules`
3. `sed -i '/\[submodule/,$d' .git/config` # careful here, worked for me since
   "[submodule" was the last stanza in my config
4. `git rm --cached themes`
5. `rm -vfr .git/modules/themes`
6. `git commit -m "Removed submodule beautifulhugo"`
7. `rm -vfr themes`

And now, since I am no a git pro, I have done the following and will just keep
my modified theme as part of the hugo site until I can figure out submodules:

1. `git clone git@github.com:thisdwhitley/beautifulhugo.git themes/beautifulhugo`
2. `rm -vfr themes/beautifulhugo/.git`
3. `git add themes`
4. `git commit -m "adding theme to this project vice as a submodule"`