---
layout: page
title: ctf-tools
permalink: /resources/ctftools
---

It's annoying having to go every time to the tools website, download the right
versions, follow the different procedures to install them, etc.

You can use https://github.com/zardus/ctf-tools to ease this boring tasks!

You can either use the provided Vagrantfile to setup an entire new VM:

```bash
wget https://raw.githubusercontent.com/zardus/ctf-tools/master/Vagrantfile
vagrant up
```

Or just clone the repository and use it in your own system:

```bash
git clone https://github.com/zardus/ctf-tools
./ctf-tools/bin/manage-tools setup

source ~/.bashrc
```
