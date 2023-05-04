setup instructions:
https://s2e.systems/docs/s2e-env.html

=== RUN DOCKER

docker run --interactive --device /dev/kvm --device /dev/kmsg -v /tmp/a:/host --tty chef-nextgen

If you want to build images yourself (not use the s2e image_build "-d" flag), you need to also include "-v /var/run/docker.sock:/var/run/docker.sock". You also need to run "sudo chgrp docker /var/run/docker.sock" after login.

=== SETUP S2E

- debian 11 (64-bit)
run with: docker run --interactive --device /dev/kvm --device /dev/kmsg -v /tmp/a:/host --tty debian:11 /bin/bash
    (x86_64)

install s2e according to the instructions (as of 2023-05-01)

- the intial repo does not work with venv (do not use it)
- you need to symlink python to python3

docker image built: chef-nextgen

next, we want to recreate the chef plugin: https://s2e.systems/docs/Howtos/WritingPlugins.html

=== TEST S2E

optional: download image - s2e image_build -d debian-11.3-i386
(debian-11.3-x86_64 is provided within the docker image)
- s2e new_project --image debian-11.3-x86_64 /bin/cat @@
- cd s2e/projects/cat
(follow instructions)
- s2e run

Useful links: https://s2e.systems/docs/FAQ.html
https://s2e.systems/docs/Tutorials/PoV/index.html
https://s2e.systems/docs/Tutorials/BasicLinuxSymbex/s2e.so.html#using-s2e-so (!)


=== PREPARE DEV ENVIRONMENT

setup clion for development inside the container: https://www.jetbrains.com/help/clion/remote-development-overview.html


=== PLUGIN DEVELOPMENT

https://s2e.systems/docs/Howtos/WritingPlugins.html
