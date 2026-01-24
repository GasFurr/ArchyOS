<div align="center">

# ArchyOS
Your Personal desktop-pure computing environment.

[![License: MPL v 2.0](https://img.shields.io/badge/license-MPLv2.0-orange)](https://choosealicense.com/licenses/mpl-2.0)
![Version](https://img.shields.io/badge/version-0.26.1--alpha-yellow)

</div>

-----

> ** Live Fast, Crash Harder.**
> This is an operating system for people, who think that "rm -rf /" is a fun starting point. We're building not for future, not for security, but for pure fun of bare-metal environments. It's basically modern DOS. "Fuck around and find out" is our motto. Break, rebuild, break again, system is built just for it. It will fail - but you will always understand why it did.

---

### Philosophy

Also, ArchyOS philosophy is based on an idea that PC will become a niche product for experts, and so, operating systems will become a decentralized space, an art form, a hobby. Being "most intuitive" and "most backwards compatible" will not be a goal anymore, because OS is not for consumers anymore, it's for creators. Consumer WILL move to portable devices or AIO out of the box ready solutions. We're trying to make computing beautiful.

It is not:
- the most optimized OS.
- the most user-friendly OS.
- the most secure OS.

If you want any of this:

- Linux.
- Windows.
- Tails OS

ArchyOS developers (and i hope users) are just someone who finds it funny to log into this OS once in a week and try to find new way to make computing FUN. Operating systems are the most overlooked part of IT fun purely because they are too complicated and designed as something you just use and don't touch. We are exploding this idea. This system is YOURS, not in "oh, it's modular and perfect fit for you" but in a "i can do anything" way. 

ArchyOS philosophical core:

1. User intent supremacy - If you wanna do something the system should not be on the way, all warnings are suggestions, not enforcement. "-i-am-an-idiot" is universal tool to do something potentially dangerous.
2. Radical pragmatism - It's easy to lose yourself building "the most \[buzzword\]" something. Our system is made 50% of parts we think is perfect already and 50% of parts we want to make better. Build what you need - destroy what sucks.
3. Hardware intimacy - DOS was hardware intimate and were so impactful because of it. It was hardware intimate from limitations of it's time. ArchyOS is intimate by design. We use modern technology and try to remove as much unnecessary complexity as we can for clean understanding of the system.
4. Determinism - System should be predictable and internally consistent, only from consistency, logic, and predictability you can make fundamentally unstable environment comfortable. If something breaks - you know why, when and how to fix it. It's behaviour should be deducible from the rules, not statistics or complex heuristics.

---

### Current Development Status

ArchyOS is currently in the **Architectural Planning / Pre-Alpha phase**.

- [x] Philosophy and Design Specification
- [ ] UEFI Bootloader Stub (Hello World)
- [ ] Archyboot Implementation...

---

### Technical Details

**Scheduling** - MuQSS:
> This is about my own beliefs in system design, i deeply agree with Kon Kolivas's vision, i think that desktop-pure system should be RESPONSIVE. Static timer-based scheduling, also, provides us with predictable, deterministic runtime for applications. BFS (Braing Fuck Scheduler) is great, but not well optimized for high core counts. MuQSS is a naturall successor to BFS ideas, optimized for modern multicore environments.
**Interfaces** - Typed System Calls
> I really thing that "everything is a file" is outdated philosophy. I think that modern operating system should have strong typed interfaces for interaction inside the OS as it gives developer more predictable environment. Classic read() and write() can lead to unexpected behaviour, which is actively destabilizing the system. Devices should have their own interfaces optimized for themselves. Also having different strong typed system calls for different interfaces in theory should make development easier.
**Display** - Direct-draw vector Surface
> I personally believe in terminal supremacy, but the way modern terminals work is a dead-end. We are destroying terminal's TTY legacy, now everything in system is directly drawing on screen without terminal grid or color codes restricting it, allowing for unprecedented levels of GUI developer's freedom.
**Configuration** - Custom declarative language Ary.
> Ary is our Shell alternative, it's a basic scripted language that is made to be predictable and human readable by using some OOP ideas. And as well - Ary, while being imperative, makes perfect sense as a declarative configuration engine. I'll explain it better later.
**Permissions** - Single-user design.
> Multi-user environments are absolutely over engineered and unnecessary for modern computing. By making system single-user i remove unnecessary complexity with UID's, GID's, Permissions, etc. ArchyOS implements capablity-based security, and so, going multi-user would be a ton of unnecessary complexity for small benefits, again, we're making PERSONAL, DESKTOP-PURE system, it will not be used for servers, there's no reason to use it for servers, and so, multi-user is just unnecessary.
**Recovery** - Archyboot subsystem.
> While ArchyOS is actively "break stuff" system - without a way to recover the system it would be unusable. And so we introduce Archyboot - basically, basic operating system as a bootloader for ArchyOS. It is monolithic and dirty, but it's stable as a stone and has all the tools to diagnose and fix problems in ArchyOS, up to basically recompiling the kernel.
**Language** - Zig.
> Zig is made as a readable language. It's designed for understanding, not fast writing. With deep understanding we can allow easy fixes for most system issues just from the fact that system itself is simple, and code is made readable, with explicit memory control and without hidden allocations. It's pretty much not far from being easily translated into machine code by a human, while not having so much critical footgun's of C and C++, and being built on modern principles. Comptime, build system, everything. Zig is just a perfect philosophical and practical match, as it's really good for bare-metal environments and really good for developers.

---

### AFHS (ArchyOS File Hierarchy Standard)

AFHS is our answer to unix FHS. Built clean, without legacy baggage and multi-user complexity. You can see for yourself - it is basically intuitive.

```text
/home/                # Home directory for the user.
/secret/              # Using capablity-based security we can make secrets visible, without making them public. This directory always needs explicit capabilites granted by user to read/write.
/driver/             # It's our userspace drivers, more about it later in "Hybrid Kernel Design" section.
/app/                # Intuitive - it's all the user apps.
/app/cache/          # It's for all the app's and app installation cache.
/temp/                # Temporary files.
/config/              # Configuration files, including Ary configs...
/mods/                # Source code of kernel modules. High-performance drivers. More in "Hybrid Kernel Design"...
/mods/propreitary/    # Basically for .a library blobs, so you can publish High-performance drivers without making their code opensource. Hi MPL-2.0;
/library/             # For all the libraries. All libraries in ArchyOS are system-wide and public.
/mnt/                 # For mountable devices like USB's, drives, etc.
/system/              # System files.
/system/info/         # Local system info, "proc/" alternative, having all machine info in file form can be quite useful.
/system/cache/        # Kernel cache.
/system/var/          # Kernel variables.
/system/src/          # ArchyOS source code.
/system/kernel/       # That's where kernel binaries lives.
/boot/                # Archyboot directory.
```

Our goal is ext4 filesystem, as it's basically peak filesystem for desktop, you don't need something better 90% of the times.

---

### Hybrid Kernel Design

So, that should be the hardest part yet, ArchyOS is a hybrid kernel. If i'm correct absolutely new type of hybrid kernel. What's the deal:
We have kernel modules `mods/` and drivers `driver/`:

- Modules are basically system libraries for kernel, their source code is contained in `mods/` directory. To update the kernel module you need to rebuild kernel. As ArchyOS is basically a microkernel it doesn't require much time or processing power, and so, absolutely rational. All files here are strongly standardized, each namefile has it's own purpose. It gives developer unprecedented levels of freedom.
- Drivers are classic userspace microkernel servers and services. They have their own Interfaces to the kernel, and basically just apps as is.

To swap out the kernel module for a userspace driver you just need to put `module.userspace("binaryname")` in module's `init()` function. Just straight up remove the internal logic of the library and call for userspace binary. Easy as that.
To swap out opensource kernel module for a proprietary one you put `module.proprietary("blobname.a")` in module's `init()` function, now it links internal logic of proprietary static library straight into kernel. Nice.

And so we get a monolithic kernel with microkernel capability. And we give user direct control over their kernel's logic. You can swap out colors in `video.zig` module, you can write your own GPU driver in `graphics.zig`, you can do anything, you are the god of your machine. That's the principle of ArchyOS - direct user control of the system in the most pragmatic and convenient way.

---

### Ary - ArchyOS shell and custom scripting language.

Now something really strange - we fully change the shell paradigm. Shell syntax is now object-oriented:
```sh
# Let's assume we need to get the processor info from shell:
> Archy Info processor
# This command will output the processor info straight to terminal.
> Archy Info processor writeInto "clipboard"
# This command will copy processor info into clipboard.
# We call Archy (basically system control center) binary, and ask for a thing it can do from class Info;
# If we just call for "processor" - it just outputs information from processor variable to the terminal.
# Or we can ask to use method writeInto on "processor" variable, as it's a universal method for this class.

# Ok, maybe we want to install an app:
> Arpack Package add "tux-race"
# We call Arpack (Archy package manager), class Package and method add.
# And then we pass an argument to method - "tux-race". All arguments are passed through "()" section after a method.
> Arpack Package add "tux-race" version "1.0"
# There we need two arguments and so - we just use two argument sections. It's an installation graph in one command.
```

It's Ary shell environment and standard ArchyOS utility package. Now you don't need to know 30 different utilities to control your system - you just need to know one and classes, everything else is intuitive and have autocompletion, so you can just `tab` your way to any problem's solution. And with this shell syntax - i think you already can see how good (or bad) Ary language is:

```sh
#Let's do an example of packages.ary config:
mainutil = "Arpack"
read_direction "fromBottom-Up."
if [package = dependency] 
-> (Arpack.Autrocorrect.Package.moveInto(config.block.dependencies))

Package.add("ArchyOS-kernel").Version("0.27.1").if [architecture = "ARM"] 
-> (
.discard
text = Archy.UI.Presets.text ("No ARM in my config", warning)
)
Package.Add("Zig-compiler").Version("1.0.0")

Ary.Block ("dependencies") 
-> (
# For autocorrect to place dependencies here
)

# 100x100 rectangle in pos 0x0 color white, alpha 0.
rectangle = Archy.UI.drawRectangle ("0x0", "100x100", "255,255,255,0")
# Will delete rectangle, then text.
Archy.UI.delete (rectangle, text);
```

It's just an example in prototyping stage, there's no full specification yet, but vision is here. It's a long way until we start implementing even a little bits of this, but it sounds like a good idea.

---

### Memory management quirks

ArchyOS doesn't have a garbage collector on kernel level as it increases complexity and makes system unpredictable. And so, our memory management approach is lifetime-based, there are the rules:
1. First is the process. Process is owner of the pointers. Pointers are a simple layer between memory and the process.
2. Every pointer has it's unique ID, two pointers of different process can point to one memory address if both processes have capability for this memory.
3. Memory is free as long as there's no pointers that point to this memory.
4. There's no dangling pointers. All pointers has their owner process, and if the process dies - pointer dies too, cleaning all the data it was pointing to (memory safety reasons)

So the scheme is:

 - Process -> Pointer -> Data.
 - If pointer dies -> data dies.
 - If process dies -> pointer dies.
 - Process can have one pointer leading to one memory sector.
 - Different processes can have pointers leading to one memory sector.
 - There's no shared pointers - memory sharing is just copies of pointers.
So if one process want to share memory with the other - it just gives it address space and a capability key for second process to create their own pointer with same capability.


---

### Branding

Almost every tech startup at this point are using the corporate "white&blue"™ aesthetic. And i don't like it.
So i'm using my own ArchyOS "Phosphorus" color palette:

- 160C28 - Midnight Violet
(Void, background.)
- E01A4F - Amaranth
(Error, Crash, Violation)
- FF8F1F - Amber
(Accent color.)
- F2F7F2 - Mint cream
(Primary text, plain white basically.)
- 247BA0 - Cerulean
(Stability, info.)

Main colors are White(mint cream) and Amber. 
ArchyOS mascot: Librarian catgirl "Archy" - a watchful eye of the system, a director of "help" utility.

---

### Documentation

Another point about ArchyOS - we prioritize documentation as a learning tool, not a cheatsheet. Documentation should answer questions "what is" and "how to" in the most useful way. Documentation should have:
1. Tutorials - "how to write a driver", "how to trace IPC", "how to create partitions". Just QnA for system use.
2. Lessons - "what is IPC", "what is MMAP", "what is syscall". Explainations of elements of the system.
3. Desctiptions - "how does MMAP structure looks?", "how IPC data struct looks?". Basically table cheatsheets for developers.

The goal here is for the system to be fully described and explained using "help" command, so just by installing the system and going through "help" you can understand the system entirely, without access to the internet.

---

### Development scheme

ArchyOS development scheme can look strange:

1. "dev-iter-*" branches, basically developer's playground, do whatever here, AI code, broken code, non-english comments, etc. It's just a playground, it's where ideas are born, tested and often discarded, just do whatever.
2. "dev" branch, it's partially safe code pushed from iterations, it's purgatory for the code where we review it, rewrite it and make it main-worthy. There all AI code from iterations should be fully rewritten by human, all comments should be translated, everything here is in active code review stage.
3. "main" branch, it's the sacred realm, there `exists only stable, reviewed code. No AI code allowed here, every symbol should be written by human. All comments should be in English. No experimentation here.

My stance on AI code is simple: it's bad. But it's just enough to start work going. AI workflow for ArchyOS is basic: Generate, Understand, Refine, Rewrite. Even if all the changes is purely cosmetic - please rewrite it by hand, i don't care if you changed nothing anyway. AI is just a statistic model, it doesn't understand what it's writing, and so, if you don't understand what AI wrote - then NOBODY on earth understands it. Be responsible about what you write.

Using AI for code itself isn't that bad, just remember - AI can't think for you, reason for you, and debug for you. It doesn't understand the machine, it can't do differential diagnosis, it has zero intuition about emergent behaviour. You ARE smarter than AI, no matter what's your level. AI has all the data. But only YOU see the logic. AI can brainstorm with you, it can't really assemble anything coherent from this brainstorming session.

> "Move towards ideal, acknowledge the limitations, embrace the suck."

---

### Versioning & Lifecycle

ArchyOS follows a dual-mode versioning strategy to balance rapid alpha iteration with stable release expectations.

1. Alpha Stage (Current): 0.YY.WW
During the Pre-Alpha and Alpha phases, versions are time-bound to the Gregorian calendar.
YY: The last two digits of the year (e.g., 26 for 2026).
WW: The week number of the year (01-52).

Example: Version 0.26.1 represents the first build of the first week of 2026.

3. Beta & Release: Major.Minor.Patch
Once the kernel reaches a "Stability Milestone" (Feature Complete), we transition to Semantic Versioning.
Major: Architectural shifts or breaking kernel changes.
Minor: New features, drivers, or system utilities.
Patch: Bug fixes and ActionID documentation updates.

---

### Legal & Licensing

All source code in this repository, unless otherwise noted, is licensed under the **Mozilla Public License, v. 2.0 (MPL-2.0)**.

---

<div align="center"> 

`> w < /* The Librarian is watching */`

**Computing as an Intent, not a Service.**

</div>
