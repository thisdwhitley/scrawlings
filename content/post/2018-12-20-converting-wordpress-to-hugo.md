---
title: Converting a WordPress blog to Hugo
#subtitle: ongoing
date: 2018-12-20
tags: ["technology","blog"]
categories: ["technology"]
---

As I've discussed in other places, I've tried quite a few times to have a blog.
That sounds pretty simple, but here are some notes on converting one of those
blogs which was a hosted WordPress blog.<!--more-->

If you do not care to read (or about) all the words and explanations, feel free
to skip to the [SHORT VERSION](#tldr)

---

### BACKGROUND

Inevitably I will be excited about a blog and sharing information for a bit of
time.  But then I'll get sidetracked or focused on the wrong thing (such as
trying to make it look awesome rather than providing awesome content).  And it
will eventually get abandoned.  In my most recent foray into blogging, I signed
up for a hosted WordPress blog.

There is some decent (probably not great) content there and when I got an email
reminding me about it and telling me I'd be charged again soon I decided that I
should just let it wither away to wherever blogs go to die.  But I want to keep
that content under the guise of retaining the information (but the reality of
being a packrat).

So I need to convert the content that is currently stored in a WordPress
database into markdown so I can "keep" it here.  *[note to future self:  explain
the reasoning behind this shift to static pages vs. WordPress]*

---

### CHALLENGE #1

There is a lot of great information online about just this type of conversion
using `pelican-import`.  Now I am not currently using `pelican` but found that
this would provide a path to where I wanted to be.  I installed everything
required per the great `pelican`
[documentation](http://docs.getpelican.com/en/latest/importer.html) but every
attempt to run the very well documented command failed with something like:  

~~~
--normalize has been removed.  Normalization is now automatic.
--parse-raw/-R has been removed. Use +raw_html or +raw_tex extension.

Try pandoc --help for more information.
Please, check your Pandoc installation.
~~~

It would create a single `.html` file and bomb so I jumped to the conclusion
that the file I was attempting to import was corrupt or not what the tool
expected.  Every page I found kept casually referring to an XML export of a
WordPress database but provided no information about how to do that or how to
generate that file.  I am going to continue that trend and only provide a link
that seems relevant:  <https://codex.wordpress.org/Tools_Export_Screen>

**Full disclosure:**  *this confusion was probably due to my ignorance around
WordPress and the guilt I found around paying BlueHost for it but never learning
more about it...*

#### SOLUTION #1

As it turns out, at the time of this writing, there is a
[Github issue](https://github.com/getpelican/pelican/issues/2322) about this.
I followed the guidance in that issue and found in
[this](https://www.hmazter.com/2018/08/move-from-wordpress-to-static-site-with-jigsaw/)
page which required that I download the `pelican` code from Github, modify the
`pelican/toosl/pelican_import.py` file, and then install using `python setup.py
install` from the directory where I downloaded the code.

**Full disclosure:**  *if these "details" seem fuzzy it is because I was really
floundering around when I got this working and did not take the type of notes
I'd prefer to be able to provide.  Please just accept my appology and read the
information in the
[link](https://www.hmazter.com/2018/08/move-from-wordpress-to-static-site-with-jigsaw/)
above...it will make all of your wildest dreams come true.*

Now you should be able to run `pelican-import` similar to the following (*I
expect to do some post-processing so I ran a pretty generic command*):

{{< highlight bash >}}
pelican-import -m markdown --wpfile -o <OUTPUT_DIR> <EXPORTED_WORDPRESS_XML_FILE>
{{</ highlight >}}

### CHALLENGE #2

The `pelican-import` utility gets me a lot closer to where I want to be.
Unfortunately, the heading is not formatted in the way that I would like it to
be.  This is also noted by Matty at
[Prefetch Technologies](https://prefetch.net/blog/2017/11/24/exporting-wordpress-posts-to-markdown/).
These observations and the way he addressed them were insightful, but I modified
it a bit for my use.  The majority of the work is in getting the "front matter"
correct.  

#### SOLUTION #2

Unfortunately, what was produced from WordPress is not consistent (...or maybe
the reality is that what *I* produced in WordPress in not consistent...whatevs).
For instance, some files have tags and others don't...I have to account for that
when I process them.  I have decided that all of these converted posts will be
tagged with "converted" "archive" and the name of the blog they came from.  So
here is roughly what my processing command looks like (and see the explanation
below the code snippet):

{{< highlight bash "linenos=inline">}}
sed -e '1s/^/---\n/' \
    -e '/Title:.*/a Subtitle: (converted post)' \
    -e 's/^Date: \(.*\) \(.*\)/Date: \1/' \
    -e '/^Slug/d' \
    -e 's/^Category: \(.*\)/Categories: ["\1"]/' \
    -e '/^Tags:/{h;s/,\s/","/g;s/:\s/: ["/g;s/$/","converted","archive","ilearnedthisthehardway"]/g};/^Status:.*/{x;/^$/{s//Tags: ["converted","archive","ilearnedthisthehardway"]/;H};x}' \
    -e '/^Status: published/c---' \
    ilearnedthisthehardway.posts/the-yard.md
{{</ highlight >}}

<!-- This text isn't converted as italic, that's just how Atom displays it. -->

Here is what that mess is doing, line by line:

1. Insert a `---` before the first line
2. Append `Subtitle: (converted post)` line after each `Title:` line
3. Remove the garbage after the date, leaving only `YYYY-MM-DD`
4. Delete the `Slug:` line since it isn't used *(I could have done this at time
   of import with `--disable-slugs`, but I didn't...)*
5. If `Category` is specified in original file (it isn't always), change it to
   be `Categories` which Hugo will recognize
6. ***This is a tricky one*** but if `Tags` are specified, I want to keep them.
   Unfortunately they are not always specified and when they are, they are in
   the wrong format.  So, if they exist, change the format to `["tag1","tag2"]`
   which Hugo understands and append the tags I want to be included.  If there
   are not tags specified initially, add the ones I want...see
   [this](https://superuser.com/questions/590630/sed-how-to-replace-line-if-found-or-append-to-end-of-file-if-not-found)
   page for a great explanation.
4. Change the `Status` line (since I don't use it) to end my front matter `---`

---

### NEXT STEPS

* I need to actually do this work...the post was largely to make note of the
  progress and commands I need to use, the work remains.

---

### WHAT I LEARNED

* Linux is fun!

---

### REFERENCE

<div id="tldr"></div>

- [Exporting Wordpress Posts To Markdown](https://prefetch.net/blog/2017/11/24/exporting-wordpress-posts-to-markdown/)
- [Move from wordpress to static site with Jigsaw](https://www.hmazter.com/2018/08/move-from-wordpress-to-static-site-with-jigsaw/)
- [sed: how to replace line if found or append to end of file if not found?](https://superuser.com/questions/590630/sed-how-to-replace-line-if-found-or-append-to-end-of-file-if-not-found)

---

### SHORT VERSION

To avoid **tl;dr** enjoy this instead:

~~~bash
## 01 Fix Pelican and build myself (lost the notes here...sorry) to convert

## 02 Process the converted files
sed -e '1s/^/---\n/' \
    -e '/Title:.*/a Subtitle: (converted post)' \
    -e 's/^Date: \(.*\) \(.*\)/Date: \1/' \
    -e '/^Slug/d' \
    -e 's/^Category: \(.*\)/Categories: ["\1"]/' \
    -e '/^Tags:/{h;s/,\s/","/g;s/:\s/: ["/g;s/$/","converted","archive","ilearnedthisthehardway"]/g};/^Status:.*/{x;/^$/{s//Tags: ["converted","archive","ilearnedthisthehardway"]/;H};x}' \
    -e '/^Status: published/c---' \
    ilearnedthisthehardway.posts/the-yard.md
~~~

---

[[return to top of page]](#)
