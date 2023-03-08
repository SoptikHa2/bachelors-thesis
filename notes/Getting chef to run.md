There are many ways of testing a program, ranging from well-known, such as unit tests, fuzzing, or QA testing, to less-known, such as property-based testing. However, those techniques either rely on a human factor correctly identifying possible edge cases and writing them correctly or pure randomness, hoping that random values will be able to discover hidden bugs.
While those techniques proved to be helpful in the past, there are still quite a few things that can be improved. Human-based techniques are expensive, error-prone, and do not tend to scale well due to the requirement of human work.
Randomness testing tends to have a low probability of hitting the right inputs to uncover an error and has to be repeated many times to provide at least some degree of confidence.
In this thesis, I will introduce a technique known as symbolic execution. This technique 
TODO: short description, short (few words) overview of results that microsoft achieved


---

TODO: Example of symbolic execution

Consider the following code:

```c
int foo(int a, int b) {
	if (a == 0) return a;
	if (a < 0) {
		b += 4;
		a += b;
		assert(a != 4);
	}
	return a + b;
}
```

TODO: Symbolic execution problems (solvers, np hardness, path explosion)

TODO: Possible mitigations (concolic execution, cutting paths that are not viable)

TODO: Results (device drivers, kernel image mounting, microsoft results, interpreters, malware analysis, ...)

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

TODO: Details on $S^2E$ modus operandi, mainly its ability to work with software without source code. Mention some examples

Chef is a  plugin allowing the debugged program to specify extra information during execution. It was built to allow building fast, easy, and correct symbolic execution engines for interpreted languages.

The key idea is not to run the target language by an SE engine but rather to run its interpreter. The interpreter being run by an existing SE engine ($S^2E$ in our case) requires some slight modifications (such as notifying the SE engine about the high-level instruction it is currently executing). And although that might result in some overhead, the result is that it is possible to reuse almost the whole code of the interpreter.

TODO: Diagram of chef + $S^2E$ cooperation

If this were not the case, we would have to build interpreters from scratch. That is a challenging, error-prone [?] task that carries a significant risk of not behaving in the same way as the original interpreter in some edge cases. Authors of Chef quoted Python Language Reference on this: "Consequently, if you were coming from Mars and tried to re-implement Python from this document alone, you might have to guess things, and you would probably end up implementing quite a different language"[?].

```pierre
Indeed, without formal semantics, the implementation might be different. 
Does Python have *official* formal semantics? No!

Some non-official attempts:

- An Executable Structural Operational Formal Semantics for Python, Maximilian Kohl, Master thesis of Saarland university
- A formal semantics of Python 3.3, Dwight Guth, Master thesis of University of Illinois at Urbana-Champaign
- An executable operational semantics for Python, Gideon Joachim Smeding, Master thesis of Universiteit Utrecht
- Fromherz, A., Ouadjaout, A., & Miné, A. (2018). Static value analysis of Python programs by abstract interpretation. In NASA Formal Methods: 10th International Symposium, NFM 2018, Newport News, VA, USA, April 17-19, 2018, Proceedings 10 (pp. 185-202). Springer International Publishing.


As they say in that last reference:

"The Python language is defined by its reference manual [21], which leaves room for ambiguity and permits implementation freedom. We base our own semantics on earlier formalization efforts [20], on the reference manual [21], and on the CPython reference implementation."

Actually, most languages do not have full formal semantics: https://v2.ocaml.org/manual/language.html 

Usually, research articles formalize a small subset + what they are adding to the language for a research paper.

What about C? There is a reference and C is arguably a small language (but still, undefined behaviours...).
```

Even if one made a great effort to ensure that the symbolic interpreter works the same way as the original one, languages tend to change quite frequently. The SE engine developers would have to make a nontrivial effort to keep it up-to-date with the latest version of the original interpreter.
With Chef's approach, the only thing that needs to be done is instrumenting the interpreter to pass some extra information to the original SE engine. The patching of the interpreter to pass the required data is relatively fast, thanks to Chef [?]. It can take just a few man-days to implement a SE engine for a new language! Authors of Chef claim that modifying the Lua interpreter took three days, and Python took five days to modify[?]. Even better, those changes tend to be relatively small, meaning updating the modified version to the newest version of the original interpreter tends to be quick or require no modifications.

However, since Chef executes the whole interpreter symbolically, there can be some extra overhead compared with hand-written SE engines. \footnote{For example, Java is cited by authors of Chef as one of the languages being too big for Chef to be used[?]}.

One of the significant challenges is often interpreter optimizations, such as hashing[?] -- used, for example, in implementations of sets or maps. As that is a nonlinear operation, an SMT solver used by SE engines[?] can take quite some time to solve those cases[?]. In situations like this, the authors of Chef recommend deoptimising the interpreter[?], such as degenerating the hashing operation always to return a constant number. This way, hashing is replaced with a linked list[?] or a similar structure, which, while significantly slower in traditional cases, can often be solved much faster by an SE engine[?].

## Getting Chef to run

Chef is quite an old software; the oldest commit in its repository[?] is eleven years old at the time of writing. The oldness of Chef resulted in quite a difficult time while trying to make it run. For example, Chef depends on $S^2E$ (v 1.0), which in turn depends on LLVM. However, because of Chef's age, the LLVM that is required is 3.2, which proved to be challenging to build on modern Linux distributions due to differences within shared libraries and modern compilers tending to behave in a bit different way. I had to use Ubuntu 13 to compile it, which had its own share of difficulties, for example, not supporting new TLS ciphers.
Running chef had similar problems, requiring me to use Debian 7, which was quite difficult to even download \footnote{After quite some time of searching, I found one remaining server where the image was hosted, and [archived](https://drive.google.com/file/d/1lISuoVg-NdT307Vpn1UOdzjdT6A4T23_/view?usp=share_link) it for future use}, let alone setup and install packages with new ciphers not supported and all the PGP keys already expired.

Before I found out which software version to use, I tried several approaches, and almost all yielded no progress. I encountered various problems, ranging from outdated compilers and libraries to qemu not working, the s2e itself segfaulting, and Chef not outputting any helpful info.

```pierre
You can use bullet lists to make the text breathe more 
```

\<APPENDIX>

The stack I'm currently running looks like this:

```
Linux host (x86) -> Docker (*) -> Qemu (Debian 7.11.0)
```
(\*): Currently running an image based on [dlsab/s2e-chef](https://hub.docker.com/r/dslab/s2e-chef/)
(to run: `xhost local:docker`, then `docker run --rm --interactive -p 5900:5900 -e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix --device /dev/kvm --device /dev/kmsg -v /tmp/a:/host --tty dslab/s2e-chef:v0.6 /bin/bash
`)

I found regular qemu and some basic system setup on the provided docker image, but I failed to find anything related to chef or s2e. Regardless, it proved very useful, as it contained just the right set of system libraries that made Chef work.

On top of the provided docker image, one has to build [chef tools](https://github.com/dslab-epfl/chef-tools), a repository with convenience scripts and utilities useful for manipulating the virtual machine state. Another thing that was needed to build was a [chef](https://github.com/S2E/s2e-old/tree/chef) \footnote{I made [several changes]((https://github.com/SoptikHa2/s2e-old/tree/chef)) to chef in order to provide more useful information to the user.}, which contains the symbolic execution engine itself.

Afterward, we want to create a VM image to run the code we want to test. I encountered an issue with this setup (as well as with every other setup I have tried), and I have been unable to run s2e-patched qemu with KVM, no matter how hard I tried.

When one reads chef instructions, it becomes apparent that running a VM under KVM is pretty helpful: we need to do the initial setup, which consists of installing the system and all useful packages & interpreters. With just software emulation, this can prove to be quite time-consuming.

The way to do so is to run the default, the one we did *not* compile, qemu. This one does work with KVM, while the s2e one does not. I strongly recommend first setting up the VM with KVM using default qemu, and only for preparation and symbolic phases, use the real s2e qemu. This has been tested to work quite well.

Another thing to pay attention to is the supported OS version; the only one I could successfully do this on is Debian 7.11.0 ([iso](https://chuangtzu.ftp.acc.umu.se/cdimage/archive/7.11.0/i386/iso-cd/debian-7.11.0-i386-netinst.iso), [pkg archive](https://archive.debian.org)).

After one sets up the VM, it can be run directly via `./run_qemu.py`  script, located in the chef tools repository.

For the convenience of future users of Chef, I prepared [a docker image](https://drive.google.com/file/d/1uhEJ7oKzvat_Y2ge4Q0GCNodk78uGFQR/view?usp=share_link) (TODO: keep link updated) that should work out of the box for everyone, with no configuration required.

### Running Chef with provided Docker Image

An example layout used for running Lua scripts:

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

The docker image uses Qemu, which spawns a window for the user to control. In order for docker to be able to do this, one has to configure the windowing system. The following instructions assume that the user is using Xorg:

TODO: concrete commands

\</APPENDIX>

## Modifying Chef to include more debug information

While trying to run Chef, I discovered that its output, while quite useful in nature, lacked debug symbols, and none of the tools provided by $S^2E$ for this purpose managed to solve the issue. I decided to modify Chef and the patched Lua interpreter. As of now, the patched Lua interpreter sends not only high-level instruction opcode and program counter but file and line numbers currently being executed as well. Chef uses this information in generated test cases and CFGs generated to notify the user of the concrete location in the program that the output relates to.

While Chef is executing the Lua virtual machine, it sends a notification called `TraceUpdate` to $S^2E$, whenever it starts processing a new high-level instruction. The notification allows $S^2E$ to know what is executed and where.

In order to send more information from Chef to $S^2E$, I had to modify the `TraceUpdate` structure. The structure included basic details about given instructions, such as its opcode and location in memory (PC). I added information about the current file, function, and line number. This provides sufficient information to the user, so they can deduce what happened and where just from the test cases and CFG alone.

TODO: Learn tikz and draw a diagram.
1) A qemu is run (all instructions are supervised by $S^2E$)
2) Inside it, lua interpreter is executed
3) A call is made for chef to start working
4) At some point, lua interpreter starts executing a high-level instruction
5) Information about the HL instruction is recorded and sent over to Chef (which is inside $S^2E$), via `TraceUpdate` (via custom instructions)
6) When a path ends, lua interpreter notifies Chef
7) Chef creates a test case with provided HL instruction debug information

While modifying Chef, I discovered that the error path bit, sent from Lua, was never actually used. I modified Chef to record it properly and fixed minor logic bugs, which resulted in Chef being able to tell apart branches that exited normally and those which exited with an error, such as failed asserts or an uncaught exception. Test cases for those branches are outputted in a new file, *err_test_cases.dat*. TODO: error path might not be used for what I think it is used

For example, with the following code, Chef will display which input triggered the failing branch and where it was.
```lua
function number_test(provider)
  local num = provider:getint(0, "num")
  if num > 4 then
    error("Expected number <= 4")
  else
    num = num + 4
  end
  return num
end
```

This code returns an error if a number exceeds 4.

Lua interpreter, supervised by $S^2E$, starts running this function. Every time it encounters a new high-level instruction, it notifies Chef, sending information such as opcode and line number.
When the interpreter executes the condition, it branches into two separate paths. One starts running the line with an error, and the other starts running the line `num = num + 4`.
After each path ends at the end of the condition, Chef is notified of this and outputs a test case for each branch, returning a concrete example of input that would trigger the execution of the branch.
The test case might look like this:
`246598 0x805984d /path/to/file.lua:4 num=>"*\x00\x00\x00"`, containing a timestamp, PC, filename, and line, and values of all symbolic variables (in this case, just `num`).