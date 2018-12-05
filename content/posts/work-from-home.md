---
title: "New 3-part tutorial! How I work from home"
date: 2018-12-03T21:37:35+01:00
draft: true
---

Doing computational research often means generating lots of data that needs to be processed and analyzed. In certain areas of molecular modelling, file sizes often go well into the gigabytes or even terabytes. As a result, downloading the full dataset to work on it locally is not only cumbersome, but inefficient. Are there any alternatives? Of course!

<!--more-->

The full series will cover three separate parts that can be used complementarily if desired.

1. [Remote file access](/tutorials/remote-data-access)
    - Access via `ssh` to your remote PC, even when it is inside a private network only accessible from an intermediate server.
    - Mount the data stored in your remote PC over SSH with `sshfs` so the software in your home computer _believe_ that the data is actually stored locally.
2. [Remote data analysis](/tutorials/remote-data-analysis)
    - Set up a Jupyter Notebook environment to perform content-rich and interactive analysis in your remote PC from home. No download involved!
3. [Remote graphical interfaces and desktop](/tutorials/remote-graphical-access)
    - Use X forwarding
    - Configure a remote graphical solution to access your graphical desktop fully, if you really need it.

While I have tried to remain field-agnostic during the three parts, I have found these guidelines to be particularly useful for molecular structure visualization and molecular dynamics trajectory analysis.

I am aware that none of this is new; just a bunch of popular tools and commands put to do what they are meant to. However, I wanted to have all that scattered documentation gathered in a single place for quick checks when I need it in the future, and thought that maybe somebody else could find it useful!

Reach me over [Twitter](https://twitter.com/jaime_rgp) or a [GitHub issue](https://github.com/jaimergp/rjaime/issues) and let me know what you think!