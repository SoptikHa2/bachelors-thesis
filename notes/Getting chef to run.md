TODO: Overview of symbolic execution. Basic idea, advantages, disadvantages, problem mitigation techniques, results achieved in the past

Resources:
https://cacm.acm.org/magazines/2013/2/160161-symbolic-execution-for-software-testing/fulltext
https://dl.acm.org/doi/epdf/10.1145/3182657
https://dl.acm.org/doi/epdf/10.1145/2654822.2541977
https://dl.acm.org/doi/pdf/10.1145/2090147.2094081
https://cacm.acm.org/magazines/2013/2/160161-symbolic-execution-for-software-testing/fulltext#R39
https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=1624015

---

# Chef

TODO: Rewrite more clearly

Chef is a plugin built upon $S^2E$ , an symbolic execution engine built upon $Klee$.

TODO: Details on $S^2E$ modus operandi, particularly it's ability to work with software without source code. Mention some examples (windows drivers without access to the source code, malware analysis).

Chef is a $S^2E$ plugin, allowing the debugged program to specify extra information during it's execution. It was built to allow building fast, easy and correct symbolic execution engines for interpreted languages.

The key idea is to not run the target language itself by an SE engine, but rather to run it's *interpreter* itself. The interpreter being run by an existing SE engine ($S^2E$ in our case) requires some slight modifications (such as notyfing the SE engine about the high-level instruction it's currently executing), but the result is us being able to reuse almost all of the code.

If this was not the case, we would have to build interpreters from scratch. That is a difficult, error-prone [?] task, that carries a significant risk of not behaving in the same way as the original interpreter in some edge cases. Authors of Chef quoted Python Language Reference on this: "Consequently, if you were coming from Mars and tried to re-implement Python from this document alone, you might have to guess things and in fact you would probably end up implementing quite a different language"[?]. (Is it ok to do this? Chef authors wrote exactly the same thing in their paper, I just copy-pasted it. But it's fitting.) Even if one undertook a great effort to ensure that the symbolic interpreter works the exact same way as the original one, languages tend to change quite frequently. Additional, nontrivial effort would be required by the SE engine developers to keep it up-to-date with the latest version of the original interpreter.
With approach that is being used by Chef, the only thing that needs to be done is instrumenting the interpreter to pass some extra information to the original SE engine. This being a rather small and easy change [?], it can take just a few man-days to implement a SE engine for a new language! Authors of Chef claim that modifying Lua interpreter took 3 days, and Python took 5 days to modify[?]. Even better, those changes tend to be quite small, meaning updating the modified version to the newest version of the original interpreter, tend to be quick, or require no modifications at all.

However, since Chef executes the whole interpreter symbolically, there can be some extra overhead compared with hand-written SE engines. For example, Java is cited by authors of Chef as one of the languages being too big for Chef to be used[?]. (TODO: move this sentence somewhere else? It disturbs the flow) One of the big challenges are often interpreter optimizations themselves, such as hashing[?]. As that is a nonlinear operation, an SMT solver that is used by SE engines[?] can be quite solve to handle those cases[?]. In situation like this, authors of Chef recommend deoptimizing the interpreter[?], such as degenerating the hashing operation to always return a constant number. This way, hashing is replaced with a linked list[?] or a simmilar structure, which, while significantly slower in traditional cases, can often be solved much faster by an SE engine[?].

## Getting Chef to run

Chef is quite an old software, the oldest commit in it's repository[?] is eleven years old at the time of writing. This resulted in quite a difficult time while trying to make it run, as Chef requires some specific versions of software to be successfully built.

I tried several approaches, and almost all of them yielded no progress at all. I encountered various problems, ranging from outdated compilers and libraries, to qemu just throwing syntax errors everywhere, qemu just not working at all, the s2e itself segfaulting, chef not outputing any useful info...
TODO: Less documentation, more text
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

For the convenience of future users of Chef, I prepared a docker image that should work out of the box for everyone, with no configuration required.

### Runnig Chef with provided Docker Image

An example layout used for running lua scripts:

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

TODO: Simple instructions on how to run the image

## Modifying Chef to include more debug information

While trying to run Chef, it was discovered that it's output, while quite useful in nature, lacked debug symbols, and none of the tools provided by $S^2E$ for this purpose, managed to solve the issue. I decided to modify Chef and the patched Lua interpreter. As of now, patched Lua interpreter sends not only high level instruction opcode and program counter, but file and line number currently being executed as well. Chef uses this information in generated test cases, as well in CFGs generated (TODO).

The process required to edit the structure *TraceUpdate*, that is sent from Lua interpreter to $S^2E$. The structure included basic deatils about given instruction, and fixed-size char array buffer (filename) and $i32$ (line number) were added to the structure in both places. Chef then saves the new content of the structure to the high level instructions, which were recorded before. Thus when generating a test case or a CFG, filenames and line numbers are available with each high level instruction.

While modifying Chef, I discovered that error path bit, sent from Lua, was never actually used by Chef. I modified it to record it properly and fixed minor logic bug, which resulted in Chef being able to tell apart branches which exited normally, and those which exited with an error, such as failed asserts or an uncaught exception. Test cases for those branches are outputed in a new file, *err_test_cases.dat*.

For example, with the following code, Chef will display which input on which line of which file triggered the failing branch.
```lua
function number_test(provider)
  local num = provider:getint(42, "num")
  if num > 4 then
    error("Expected number <= 4")
  else
    num = num + 4
  end
  return num
end
```
TODO: This paragraph is written in a really bad way, rewrite

This code simply returns an error, of a number is greater than 4. By default, a value *42* is used, but Chef outputs test cases for both branches of the *if* condition.

An output that looks like this will be written to the *err_test_cases.dat* file:
`246598 0x805984d /path/to/file.lua:4 num=>"*\x00\x00\x00"`, containing timestamp, PC, filename and line, and values of all variables (in this case, a star has acii value *42*, so the variable *num* will be set to value *42* in this case.