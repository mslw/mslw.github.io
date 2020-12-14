---
layout: post
title: Docker pruning
author: Michał Szczepanik
tags: ['software', 'tips & tricks']
---

Sometimes (especially after I have been messing around with many different containers) I get the „No space left on device” error when doing even simplest tasks (e.g. running heudiconv or trying to build a new image).
This issue _may_ be MacOS-specific.
Finding a clear answer wasn’t straightforward, so below I summarise what I learned.

The problem boils down to the Docker’s disk image, located (on MacOS) under:
```
~/Library/Containers/com.docker.docker/Data/vms/0/Docker.qcow2
```

See  [Stackoverflow: What is Docker.qcow2](https://stackoverflow.com/questions/49887747/what-is-docker-qcow2) for details.
In short, this is a virtual disk, with a size cap set to 64 GB by default.
Different things are written there, and not always cleaned, causing it to grow in size (also, I believe that for complex reasons, the space taken on your disk does not necessarily reflect the size of the contents).
Docker is conservative when it comes to removing things.
If it gets filled to the brim, your containers can’t work properly.

For a solution, read  [Docker docs: Prune unused Docker objects](https://docs.docker.com/config/pruning/).
You can clean up some space by running:
```
docker system prune
```
or, more likely (a thorough version - remember that it will destroy data stored in containers!):
```
docker system prune --images
```

Alternatively, the _burnt ground_ strategy would be to remove the Docker.qcow2 file altogether (as it will rebuild next time docker is used), but it seems unnecessary.

Text initially published on my lab's forum, now nonexistent.
