---
layout: post
title: "Python, Windows, and how times are changing"
author: Michał Szczepanik
tags: ['python']
date: 2020-04-01
---

I started using Python in 2011, as part of my BSc curriculum in neuroinformatics (the introductory course was part of the first semester IIRC). The Faculty was running Fedora on all computers, and so was I on my personal laptop. Back then, the advice to people who worked on Windows was to try and avoid that, because things with Python and Windows were complicated.

The common Python on Windows advice soon changed to "install anaconda". This distribution, complete with a set of popular scientific packages (or a simple way to install them) had its initial release (0.8.0) in July 2012, according to Wikipedia.

In practical terms, many of the issues with Windows boiled down to installation of two commonly used packages, numpy (for numerical operations with n-dimensional arrays) and scipy (with further numerical routines, including signal processing, linear algebra and statistics). While on Linux a simple `pip install numpy scipy` was sufficient ever since I can remember, on Windows it wasn't so simple (_If you're on Windows, and you try the same thing with pip, all hell will break loose_ - stated [this page from numpy wiki](https://github.com/numpy/numpy/wiki/Whats-with-Windows-builds), which also offered detailed explanation). In short, both libaries rely on code written in C and Fortran and there was no agreement on which compilers should be used for Windows (with implications for performance). As a result, no compiled versions (published by the development teams) were available for installation with pip. Anaconda provided those within their ecosystem.

However, that's not to say that it was the only option to get numpy/scipy working on Windows. Respected, yet unofficial, binaries (_wheels_ in Python nomenclature), were provided by Christoph Golke from Laboratory of Fluorescence Dynamics, UC Irvine on [his university website](https://www.lfd.uci.edu/~gohlke/pythonlibs/). So you could download them manually and install with `pip install numpy-<version>.whl` (and repeat for upgrades). Very convenient, though I have a feeling that for most people going to some (however respected) person's university website was less appealing than _just installing_ anaconda.

Things changed around 2016, when Numpy team began releasing Windows binaries to [pypi](https://pypi.org/) (Python Package Index, default source for installation with pip). I can't quickly find the exact release which introduced those binaries; however in the release notes for 1.11.0 (March 2016) I found this annotation: _No Windows (TM) binaries are provided for this release due to a broken toolchain_. 

Another milestone came in October 2017, when the scipy team decided that their project reached sufficient maturity for the version number to be changed from 0.x to 1.0. The [release notes](https://docs.scipy.org/doc/scipy/reference/release.1.0.0.html) started with:

> We are extremely pleased to announce the release of SciPy 1.0, 16 years after version 0.1 saw the light of day. It has been a long, productive journey to get here, and we anticipate many more exciting new features and releases in the future.

Publication of official Windows wheels was one of the developments leading up to the 1.0 release. As a result, `pip install numpy scipy` now works seamlessly on Windows.

What's more, developments came on the side of Windows as well. First, [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) was introduced, making it possible to literally run Linux on Windows (with bash and all command line goods). And very recently, Microsoft introduced [Windows Terminal](https://github.com/Microsoft/Terminal), a new, [shiny](https://www.youtube.com/watch?v=8gw0rXPMMPE) command line interface (which can be used to run cmd, powershell, bash, etc.) with tabs and customisation.

The anaconda distribution definitely a huge appeal, in that it provides not only raw Python, but also a collection of packages and an IDE, as well as methods to manage them (conda, with its own kind of virtual environments), and in [some cases](https://www.anaconda.com/tensorflow-in-anaconda/) significant performance optimizations. At the same time, I feel that the main reason conda was being recommended (ease of installation) is not the right reason any more.
