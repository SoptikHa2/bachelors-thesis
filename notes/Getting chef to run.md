Chef seems to be a quite fragile software, requiring careful configuration of system and libraries, and a bad distro version can mess everything up.

I tried several approaches, and almost all of them yielded no progress at all. I encountered various problems, ranging from outdated compilers and libraries, to qemu just throwing syntax errors everywhere, qemu just not working at all, the s2e itself segfaulting, chef not outputing any useful info...

The stack I'm currently running looks like this:

```
Linux host (x86) -> Docker (*) -> Qemu (Debian 7.11.0)
```
(\*): Currently running an image based on [dlsab/s2e-chef](https://hub.docker.com/r/dslab/s2e-chef/)
(to run: `xhost local:docker`, then `docker run --rm --interactive -p 5900:5900 -e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix --device /dev/kvm --device /dev/kmsg -v /tmp/a:/host --tty dslab/s2e-chef:v0.6 /bin/bash
`)

On the provided docker image, we can find regular qemu, and some basic system setup, btu I failed to find anything related to chef nor s2e. Regardless, it proved to be very useful, as it contained just the magical set of system libraries, that made it work.

On top of the provided docker image, one has to build [chef tools](https://github.com/dslab-epfl/chef-tools). Chef tools and [chef](https://github.com/S2E/s2e-old/tree/chef) provide everything we need. (Note: I have a [fork](https://github.com/SoptikHa2/s2e-old/tree/chef) that provides more info to the user)

Afterwards, we want to create a VM image that will run the code that we want to test. I encountered an issue with this setup (as well as with every other setup I've tried), and I've been unable to run s2e-patched qemu with kvm, no matter how hard I tried.

When one reads [chef instructions](https://github.com/dslab-epfl/chef-tools#setting-up-the-chef-vm), it becomes apparent, that it's quite useful to run a VM under kvm: we need to do the initial setup, which consists of installing the system and all useful packages & interpreters. One does *not* want to do this with software emulation.

The way to do so is to run default, the one we did *not* compile, qemu. This one does work with KVM, while the s2e one does not. I strongly recommend to first set up the VM with KVM using default qemu, and only for preparation and symbolic phases, use the real s2e qemu. This has been tested to work quite well.

Another thing to pay attention to is the supported OS version, the only one I could successfully do this on, is debian 7.11.0 ([iso](https://chuangtzu.ftp.acc.umu.se/cdimage/archive/7.11.0/i386/iso-cd/debian-7.11.0-i386-netinst.iso), [pkg archive](https://archive.debian.org)).

After one sets up the VM, it can be run directly via `./run_qemu.py`  script, located in the chef tools repository.

---

An example layout (using lua):

Docker (Ubuntu 14.04):
```
~/s2e
    | s2e - source code[1]
    \ build - build dir [2]
    
~/tools - chef-tools [3]

~/vm
   | chef_disk.s2e - 1st step configuration (kvm)
   \ chef_disk.s2e.1 - prepared image (prep phase)
```

1) [s2e, branch chef (custom fork)](https://github.com/SoptikHa2/s2e-old/tree/chef)
2) create build dir, run `ln -s ../s2e/Makefile`, run `make`
3) [chef-tools](https://github.com/dslab-epfl/chef-tools), containing run scripts and configuration

Qemu (Debian 7.11):
```
~/lua [4]
```
4) [lua, altered to work with chef (custom fork)](https://github.com/SoptikHa2/chef-symbex-lua), run via `chef/lua_runner.py`
---
