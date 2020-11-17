---
layout: post
title: Giving in to pyenv
author: Michał Szczepanik
tags: ['software', 'python', 'tips & tricks']
---

There are several ways of installing and managing python on macOS, and pyenv is one of them. Below I explain why I was avoiding it for a long time and why I gave in. Following is a short reference with how I currently set things up and some most useful commands.

Before I start: there are, naturally, many blog posts about using pyenv. However, most of what I found didn't cover my use case entirely, so here is a compilation of things I wish I knew earlier.

## Part 1: what I used before

For a very long time I have been using [homebrew](https://brew.sh/) ("the missing package manager for macOS") to take care of my python installation. I also use homebrew to install many other programs, so `brew install python3` and, if needed `brew update python3` were nice, easy and natural.

Another key element was [virtualenv](https://virtualenv.pypa.io/en/stable/) ("a tool to create isolated python environments"). Install python with homebrew, create per-project virtual environments with virtualenv, have a relatively useful set of packages in the global installation and be happy. No need for extensions, such as [virtualenvwrapper](https://pypi.org/project/virtualenvwrapper/).

## Part 2: the turning point

While homebrew-installed python is easy to update, it is perhaps a little bit too easy. Sometimes, I triggered an update when installing or updating some other program (formula). Additionally, default behaviour of homebrew is to perform cleanup, that is remove old versions of a program after installing a new one. This is desired in most cases (to avoid eating up disk space), but not always with python.

This is because virtualenv does not create a copy of python installation, instead relying on the existing one (at least by default, I should check what the `--copies` / `--always-copy` option does). As a result, upgrading python through homebrew breaks links to python used in a virtual environment. And if an older version of python is needed when building an environment, then it somehow needs to be installed in the first place.

So from time to time I had a bad day, because an update broke my python. In most cases, however, I could rebuild any environment (with the new version of python) simply by calling `virtualenv path/to/the/env` and running `pip install -r requirements.txt` inside. After all, there isn't that much difference between, say, 3.7 and 3.8.

However, some changes are game breaking. As of today, atlasreader does does not work on python above 3.7 (I think due to changes in packages on which it depends, but I didn't investigate). This is how I ended with python 3.9 and needing 3.7.





