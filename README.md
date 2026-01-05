<div align="center">

# ArchyOS
**The Desktop-Pure Computing Environment.**
*IBM Industrial Design. Bauhaus Geometry. Zig Performance.*

[![License: MPL v 2.0](https://img.shields.io/badge/license-MPLv2.0-orange)](https://choosealicense.com/licenses/mpl-2.0)
![Version](https://img.shields.io/badge/version-0.26.1--alpha-yellow)

</div>

-----

> **⚠️ Live Fast, Crash Harder.**
> ArchyOS is built on a radical contract: **the kernel must never fail unexpectedly.** All crashes and fails should be directly tied to user mistakes. This stability is what makes true experimentation possible. "Live Fast" isn't about instability - it's about the freedom to modify, break, and rebuild your own space with velocity and precision, knowing the foundation beneath you is solid. Here, a crash isn't a mystery; it's a logged, traceable, and deliberate consequence. We're using `rm -rf /` as a fun starting point to build something different, and so we embrace our mistakes.

---

### Philosophy

ArchyOS is a fundamental rethinking of the OS, built on a non-negotiable contract defined by four interlocking principles. This contract separates absolute kernel stability from sovereign user-space freedom:

1. **Radical Pragmatism** - Unnecessary complexity is eliminated while effective solutions are preserved. Legacy abstractions that hinder performance are removed. 

2. **Hardware intimacy** - We target a 2015+ baseline (UEFI, NVMe, 64-bit multicore). Nothing past that considered "legacy" - it's just obsolete, and so it's not enabled by default or can't be supported technically. System is DESIGNED only for modern hardware.

3. **Transparent Determinism** - System should be totally predictable and internally consistent. It's behaviour should be deducible from the rules, not statistics or complex heuristics.

4. **Intent Supremacy** - User intention is the only source of legitimacy in the system, everything beyond the user's control must be absolutely stable; any error must be directly due to user error, not to internal violations.

One thing should be clear our intention is not "cybersecurity" or "immutable system" or some other buzzword. Our intention is to make system transparent and understandable. If you can see through your system and understand what is happening you are more safe than if you have million dollar heuristics algorithm.

Want real security?

1. **Physical:** Don't let strangers use your computer
2. **Network:** Unplug Ethernet / turn off WiFi
3. **Software:** Don't run untrusted code
4. **You:** Don't be stupid

ArchyOS tools help with 1-3. Nothing can help with #4.

Also, ArchyOS philosophy is based on an idea that PC will become a niche product for experts, and so, operating systems will become a decentralized space, an art form, a hobby. Being "most intuitive" and "most backwards compatible" will not be a goal anymore, because OS is not for consumers anymore, it's for creators. Consumer WILL move to portable devices or AIO out of the box ready solutions. So our goal is not to create a grannyOS or hackerOS, it's to make a hobbyOS where operating system that is a hobby for the user. We're trying to make computing beautiful.
Although i respect modern osdev community, i think it's too focused on creating a technodemo's like "UNIX clone in Rust" or "Microkernel on Haskell", it's cool, but there's rarely something really innovative in it.

---

### Current Development Status

ArchyOS is currently in the **Architectural Planning / Pre-Alpha phase**.

- [x] Philosophy and Design Specification
- [ ] UEFI Bootloader Stub (Hello World)
- [ ] Archyboot Implementation...

---

### Technical Details

| Problem | Tech | Reasoning |
| :--- | :--- | :--- |
| **Scheduling** | Timer-Based (MuQSS) | Static time slices; predictable execution without heuristics or overhead. |
| **Filesystem** | ARFS (Atomic 4MiB) | NVMe-optimized extents. Fixed-size metadata for zero-parsing/SIMD speed. |
| **I/O Strategy** | Direct Extent Mapping | Kernel bypass; process receives LBA-pointer + Capability for direct HW access. |
| **Small Storage** | Sub-extent Catalogs | One 4MiB extent = 1024 x 4KiB slots. High-density storage for configs and notes. |
| **Metadata** | Self-Describing Header | First 4KiB of a file contains its identity/tags; allows re-indexing from raw disk scan. |
| **Memory** | Child-Pointer Model | Kernel-tracked pointer lifetimes per PID; atomic reclamation on process exit. |
| **Architecture** | Hybrid (Type-A/B) | Type-A: Isolated microkernel servers. Type-B: Statically linked high-speed modules. |
| **Hierarchy** | AFHS (Flat Labels) | Simple 3-symbol disk labels (H:, R:, B:) and explicit mount points (home/, root/, boot/). |
| **Interfaces** | Typed System Calls | Rejection of byte-stream abstractions; syscalls match specific hardware data types. |
| **Display** | Vector Surface | Direct framebuffer draw; rejection of TTY legacy for geometry-based UI. |
| **Configuration** | First-Class TOML | System state is a static reflection of the config; eliminates side-effects. |
| **Observability** | 8-bit ActionID | Registry of 256 codes; maps every failure to a specific documentation index. |
| **Recovery** | Archyboot Monolith | Monolithic fallback kernel; capable of re-building ARFS index and system repair. |
| **Language** | Zig | Manual memory control, no hidden control flow, designed to be radable, perfect philosophy match. |

## Hybrid Kernel Design

The system utilizes a microkernel core with specialized high-speed interfaces for hardware communication. Drivers are categorized by their level of integration and risk:

* **Type-A Drivers:** Standard microkernel "servers." These run as isolated processes in their own memory space. They are safe, swappable, and communicate via typed system calls.
* **Type-B Drivers:** Statically linked modules with direct hardware access. These are high-performance "blackboxes" linked at the kernel level for speed-critical paths. 
    * **Security:** Type-B modules are considered a security threat. 
    * **Verification:** All Type-B modules must carry a digital signature. Unsigned modules require the `--i-am-an-idiot` flag to be linked, acknowledging the risk of kernel-level instability or compromise.

---

## AFHS (ArchyOS File Hierarchy Standard)

Storage is organized into specific root-level functional directories, mapped via disk labels.

```text
H:home/             # Sovereign user workspace
R:secret/           # Encrypted system/user secrets
R:drivers/          # Type-A driver binaries
R:apps/             # User applications
R:temp/             # Volatile scratch space
R:config/           # Global TOML system configuration
R:mods/             # Signed Type-B kernel modules
R:library/          # System shared resources/blobs
R:drive/            # Internal storage mount points
R:mnt/              # External/Removable media mount points
R:system/           # Core microkernel binaries
B:boot/             # Archyboot recovery and bootloader.
```

## ARFS Disk Labels

Drives and partitions use a 3-character naming convention (Capital letters and numbers only). Because the system is desktop-focused, it supports a hard limit of 99 concurrent drives, prioritizing clarity over infinite scalability.

| Label | Mount point | Content |
| --------------- | --------------- | --------------- |
| B: | boot/ | Archyboot monolithic Fallback |
| R: | root/ | System core, Drivers, Apps |
| H: | home/ | Sovereign User Workspace
| D[1-99]: | drive/ | internal storage drives |
| M[1-99] | mnt/ | Removable media (USB/External) |

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
3. "main" branch, it's the sacred realm, there exists only stable, reviewed code. No AI code allowed here, every symbol should be written by human. All comments should be in English. No experimentation here.

My stance on AI code is simple: it's bad. But it's just enough to start work going. I, myself, a 16 y.o., mostly learning while i'm writing the code, and sometimes it's better to make something with AI, understand it, and then rewrite, than do nothing. But if you don't understand AI code - this code is bad by default, even if it works perfectly. AI workflow for ArchyOS is basic: Generate, Understand, Refine, Rewrite. Even if all the changes is purely cosmetic - please rewrite it by hand, i don't care if you changed nothing anyway. AI is just a statistic model, it doesn't understand what it's writing, and so, if you don't understand what AI wrote - then NOBODY on earth understands it. Be responsible about what you write.
AI for writing docs? Sucks. But again: Generate, Understand, Refine, Rewrite. If without AI you can't explain your own code - it's a bad code. Quality control for documentation should actually be even more strict, than quality control for the code, but if it gets work done - just use standard workflow, never even try to push AI code and docs anywhere without reading it first.
Did i used AI for this readme? Yeah, in some part. But all the design choices and philosophy are purely mine, i used AI only to give all this thought a comprehensible form, to get MY work done, not to get work done FOR ME. 

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
Once the kernel reaches a "Stability Milestone" (Feature Complete), we transition to Semantic Versioning (SemVer).
Major: Architectural shifts or breaking kernel changes.
Minor: New features, drivers, or system utilities.
Patch: Bug fixes and ActionID documentation updates.

### Some Q&A

> Most hobby OSes fail from lack of applications. Your driver model needs to make porting software easier, not harder.
1. About just that - ArchyOS targets to developers, exactly developer's who want to go deep, and so, it's designed to have documentation that can be used as a self-learning material, you can learn how to develop drivers for ArchyOS just by reading the docs.
> Timer-based scheduling (MuQSS) is predictable but may waste CPU. Modern schedulers (CFS) use heuristics because they work well for unpredictable interactive workloads.
2. CFS was made for servers, to be "fair" - read Kon Kolivas' work on BFS. I think desktop-pure system should be RESPONSIVE. Responsiveness is our first goal, and so all "heuristics" we need - is just giving priority to the focused application, to the application user uses right now, and separate thread just for the media, because media is really important in modern day, i can't see my life without music, and i hate when music starts lagging while i do a big compile.
> "All crashes are logged and traceable" is a massive promise.
3. Nothing complicated - just log with all system interactions. ActionID is basically a name tag for all data going through the system, every IPC, memory allocation, etc. And so opening the log you can see exact last operation, who sent it and who tried to read it. Nothing complex, theese 8 bit's are droplet in the sea, they will make close to no difference in modern hardware, but makes debugging and logging 1000x times more easier.
> The "pure desktop" focus might lock you into x86_64 while the world fragments. What about ARM, RISC-V?
4. It's a problem of the world, system is not tied to x86_64 - you can just go and cross-compile it, and if i ever see that desktop is moving onto other platforms - i will adapt, but for now, x86_64 is the only viable "open Architecture" system, the only Architecture where everything is swappable, standardized and is just working, most ARM projects are AIO closed-source shit, and RISC-V is too premature, i can't see future of it, i can't be sure that open Architecture will be mainstream for RISC-V and i see that hardware support for it gonna be fucking bad actually, it's not standardized, scattered and immature, that's all of it.
> Building for the future is a massive bet.
5. Yeah, i can't be sure that my vision is the only truth, again, i can't see future. But i believe in my vision, and either way, even if i am wrong - it's good to build something different and new in a pretty stagnant niche, and as Linus said - You should be naive to even start project as big as operating system. If my bet is wrong - i will just move on, it's not some holy war, i just want to try to build something different.

---

### Legal & Licensing

ArchyOS is a multi-layered project. We use different licenses to ensure the code remains open, the knowledge remains shared, and the identity remains protected.

#### 1. The Engine (Code & Logic)

All source code in this repository, unless otherwise noted, is licensed under the **Mozilla Public License, v. 2.0 (MPL-2.0)**.

* **Why:** It allows for "Type-B" modules to be linked without forcing a total system license change, while ensuring any improvements to the ArchyOS core are contributed back.
* **Key Requirement:** Modifications to MPL-licensed files must be made available under the same license.

#### 2. The Library (Documentation & Lessons)

All documentation, tutorials, and "Lessons" are licensed under **Creative Commons Attribution-ShareAlike 4.0 (CC BY-SA 4.0)**.

* **Why:** Knowledge should be a permanent resource. You can remix and share our docs, but you must credit ArchyOS and share your version under the same terms.

#### 3. The Identity (Branding & Mascot)

The ArchyOS name, the "Phosphorus" color palette, and the mascot **"Archy the Librarian"** are licensed under **CC BY-NC-ND 4.0**.

* **Why:** You are free to share her image and promote the project, but you cannot use her for commercial purposes or modify her design without explicit permission. She is the mascot of the system, and her identity stays with the project.

---

<div align="center"> 

`> w < /* The Librarian is watching */`

**Computing as an Intent, not a Service.**

</div>
