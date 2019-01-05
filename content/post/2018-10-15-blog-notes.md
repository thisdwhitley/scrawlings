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
  - this looks non-trivial also...
- small "share" icons
- similar posts (based on tags) = this is actually pretty easily accomplished by putting "showRelatedPosts = true" in config.toml [I may want to change the style though...]
- categories
- docker social page
- table of contents (this is harder than it should be)
  - here is a page with some hints:  <https://github.com/gohugoio/hugo/issues/1778#issuecomment-313895910>
  - and here is how hugo says to do it (which I can't get to work but it looks like this is what pad is doing):  https://gohugo.io/content-management/toc/
  - I like this one but I have no idea how to get this to work:  <https://disaev.me/p/blog-engines-and-static-site-generators/#table-of-contents>
  - I need to figure out how to use these partials, I think it'd be good so that I could set a variable in the header to include the disclaimer only when it is needed
- take a look at directory structure.  if all my posts go into "posts" that's going to tough to look at.  It looks like pad does it as "year/month/title/" and potentially puts images in there too?
  - if I do this, I can either create a generic `_index.html` file in the directories *OR* I can add this structure as a taxonomy in `config.toml` and then I can browse just to thisdwhitley/2018/04/ to see all the posts from April 2018
- fix "Short Version" and "[return to top]" and consider putting this into `template` or `shortcode`?

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
