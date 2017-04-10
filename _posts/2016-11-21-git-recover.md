---
layout: post
title:  "Git recover"
date:   2016-11-10 23:03:00
author: anuj
categories: Technical Git
comments: true
---

Over the weekend I worked on a small Android app. Since it was a small project and I was mostly working alone, I did not care to add version control. Once it was finished, I decided to push it to GitHub.

Up comes the terminal,

```Bash
$ git init
$ git add .
```
That is when I realized I had not updated the default `.gitignore`. Happens all the time, no biggie.

I go ahead and update the file, and in a moment of reckless urgency, type

```Bash
$ git rm -rf .
```
instead of

```Bash
$ git rm -r --cached .
```
thereby deleting all my work, f*@#! I had a feeling that since I had at least added the files to the git index, there might be a copy somewhere. Turns out, there was.

```Bash
$ git fsck --lost-found
```
recovered all my files into `.git/lost-found/other`, yay!. The problem was, that the filenames didn't make a lot of sense to me `548d4722c20c7236e60d6f7ace636032b2954036`, and I didn't have a file tree in the index.

In comes python. A short script to match all the files that start with our package name, and I had all of my classes.

```Python
import os, glob

for filename in glob.glob("./*"):
    searchfile = open(filename, "r")
    firstline = searchfile.readline()

    if "com.moldedbits.chip" in firstline:
        print filename
        continue
```
Small modifications to the script got all the resource files as well, and the day was saved.

Happy coding!

The moldedbits Team

{% if page.comments %}
{% include disqus.html %}
{% endif %}
