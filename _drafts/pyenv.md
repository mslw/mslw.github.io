---
layout: post
title: Giving in to pyenv
author: Michał Szczepanik
tags: ['software', 'python', 'tips & tricks']
---

There are several ways of installing and managing python on macOS, and in this post I am taking a closer look at pyenv. The post is divided into two segments. In the first (which I called "life story" during writing) I explain why I was avoiding pyenv for a long time and what caused me to give in. The second is technical and provides a short reference for creating and using my current setup.

There are, obviously, many blog posts about using pyenv, including ones from [Real Python](https://realpython.com/intro-to-pyenv/), [Towards Data Science](https://towardsdatascience.com/managing-virtual-environment-with-pyenv-ae6f3fb835f8) or the one I liked a lot from [Raycent](https://raycent.medium.com/managing-python-on-macos-the-clean-way-7673cab874f6). However, most of what I found didn't cover my use case entirely, so here is a compilation of useful tips (which I wish I had known earlier).

## Introduction
### What I used before

For a very long time I have been using [homebrew](https://brew.sh/) ("the missing package manager for macOS") to take care of my python installation. I also use homebrew to install many other programs, so doing `brew install python3` and, if needed `brew update python3` were nice, easy and natural.

Another key element was [virtualenv](https://virtualenv.pypa.io/en/stable/) ("a tool to create isolated python environments"). Install python with homebrew, create per-project virtual environments with virtualenv, have a relatively useful set of packages in the global installation and be happy. No need for extensions, not even the popular [virtualenvwrapper](https://pypi.org/project/virtualenvwrapper/). This has served me for quite a while.

### The turning point

While homebrew-installed python is easy to update, it is perhaps a little bit too easy: sometimes, I would trigger an update when installing or updating some other program. What's more, the default behaviour of homebrew is to perform cleanup, that is to remove old versions of a program after installing a new one. This is all well and good in most cases (to avoid eating up disk space), but not necessarily with python.

The problem is that virtualenv does not create a full copy of a python installation; instead, it relies on the existing one (at least by default, I am not quite sure what the `--copies` / `--always-copy` option does). As a result, upgrading python through homebrew will break links to python used by a given virtual environment. And aside from breaking existing environments, if an older version of python happens to be needed to build one, then it somehow needs to be installed in the first place.

So from time to time I had a bad day, because an update broke my python. In most cases, however, I could rebuild any environment (albeit with the new version of python) simply by calling `virtualenv path/to/the/env` and running `pip install -r requirements.txt` inside. After all, there isn't that much difference between, say, 3.7 and 3.8 (and it's always nice to have shiny new things, like the [walrus operator](https://www.python.org/dev/peps/pep-0572/#abstract)).

However, some changes are game breaking. As of the day of writing, [atlasreader](https://github.com/miykael/atlasreader), an awesome tool for generating coordinate tables and plots for brain images, does does not work on python versions above 3.7 (I think due to changes in packages on which it depends, but I didn't investigate).

And this was a story of how I ended with python 3.9 while needing 3.7.

### Assumptions

The recommendations below will be most useful for people who, like me:
1. work on several (potentially drawn-out) projects simultaneously
2. rely on some standalone python programs, which may not always be quickly brought up to date
3. want to use jupyter lab

and for these reasons need to have both several versions of python and several sets of python packages (virtual environments) built for a given python version.

To this end, pyenv provides a unified, high-level interface for simple installation and swapping out both python versions and specific virtual environments - and has become my tool of choice.

## The technicalities

Instructions below are based on MacOS with zsh and homebrew. The specifics will be different, but pyenv will work on other shells and OSs.

### Installation - pyenv

When using homebrew (though other ways also available), the installation is straightforward. Pre- and post-installation steps are listed in [pyenv's readme](https://github.com/pyenv/pyenv#homebrew-on-macos) on github, and can be summarised in three steps:

1. Instal dependencies if not present already (xcode tools, openssl and the like).
2. Install pyenv itself: `brew install pyenv`
3. Add pyenv init to the shell by editing the `.zshrc` (or equivalent) file. More on that later.

Note: if python was previously installed with homebrew, it does not need to be removed: the pyenv-installed python lives entirely separately from the homebrew-installed one. I did remove it on my desktop (which involved disassembling a tangle of dependencies one by one), but in hindsight it was a waste of time.

After pyenv, it is good to install [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv), a pyenv plugin which provides features to manage virtualenvs, with `brew install pyenv-virtualenv`. The readme suggests making another addition to `.zshrc` (pyenv virtualenv-init, to enable auto-activation of environments), which I personally skipped.

Unfortunately, pyenv does not mix smoothly with homebrew. When things are left alone, `brew doctor` will complain about existence of python config files created by pyenv, and, if I understand correctly, python dependencies from homebrew would use the pyenv-managed python. According to recommendations [here](https://raycent.medium.com/managing-python-on-macos-the-clean-way-7673cab874f6) and [here](https://github.com/pyenv/pyenv/issues/106), the best way to deal with it is by adding an alias for homebrew to `.zshrc`. This should make homebrew oblivious to pyenv-installed python (and the homebrew-installed python will be used as a dependency for everything else homebrew does). While this means having two separate python ecosystems, seamless operation far outweighs the redundancy.

In the end, here are the `.zshrc` additions: one for pyenv, one for pyenv-virtualenv, and one for aliasing homenrew.

```zsh
if command -v pyenv 1>/dev/null 2>&1; then
	eval "$(pyenv init -)"
	eval "$(pyenv virtualenv-init -)"  # OPTIONAL
	alias brew='env PATH="${PATH//$(pyenv root)\/shims:/}" brew'
fi
```

### Organising things with pyenv

With pyenv installed, I think this is a reasonable way to proceed:
* install the latest python version and set it as global
* install other versions, needed for virtualenvs or standalone programs
* install jupyter lab under global
* for each project that uses jupyter, install an ipykernel (see below)
* for each standalone, switch to the needed python version before installation or usage

### Organising things for jupyter

Pyenv or not, the right way for working with virtual environments is to install jupyter lab only once, globally (that is under your primary python version), and then only install _kernels_ for each environment you want to use with jupyter ([docs here](https://ipython.readthedocs.io/en/stable/install/kernel_install.html)). To install a project-specific (virtualenv-specific) kernel:

* Activate virtualenv
* Install the ipykernel package inside: `pip install ipykernel`
* Register a new kernel: `ipykernel install --user --name myenv --display-name "Python (myenv)"`
* Deactivate

Kernels can be listed with `jupyter kernelspec list` and, if no longer needed, removed with `jupyter kernelspec remove`.

### Usage: pyenv commands

The list of pyenv commands is available [here](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md) and can be displayed with `pyenv commands`. For me, these are the most important:

* `pyenv shell x.y.z` to use python x.y.z in the current shell
* `pyenv activate <name>` to manually activate a pyenv virtualenv
* `pyenv whence <some executable>` to see which environment has that executable (e.g. atlasreader)
* `pyenv version` to see which version is currently active
* `pyenv versions` to see versions available
* `pyenv help <command>` to get help for a command
