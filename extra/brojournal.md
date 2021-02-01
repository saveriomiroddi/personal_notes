## Mon 01/Feb/2021

- Admin/tools
  - [x] Clonezilla: manual run (restore) via tools, without boot
    - [x] Fix issue with dumps to be restored by the Focal clonezilla version
  - [x] Git
    - Fight `format-patch` accidental misconfiguration

- Projects/Open source
  - QEMU research paper
    - Scripts
      - [x] Add SMT support
      - [x] Create `run_all.sh` script
      - [x] Rename plotter `--compare` to `--scale`
    - [x] Test on bare Ubuntu Server
      - [x] Minor fixes
      - [x] Review results
    - [ ] Dive into the QEMU RISC-V implementation
      - [x] Find out if there is a core/hw_thread concept
      - [x] Summarize and send

## Sun 31/Jan/2021

- Studies
  - Other
    - [x] Python for text processing, instead of Perl
    - [x] Patch/diff, instead of Perl
  - Gnuplot
    - [x] Multiplots
      - [x] Handling diagrams with different x ranges

- Projects/Open source
  - QEMU research paper
    - [x] Scripting/features
      - [x] Quick cleanups/refactorings
      - [x] Informal HT local tests
      - [x] Add SMT disabling
      - [x] Add support for local (host testing)
      - [x] Add support for normalized/comparison diagram
    - [x] Personal workstation testing (host+guest)
      - [x] Base testing
      - [x] Pinning testing
        - [x] Review taskset+isolcpu
          - [x] Read resources
            - [x] https://stackoverflow.com/questions/14512197/what-does-taskset-in-linux-exactly-do
            - [x] https://www.codeblueprint.co.uk/2019/10/08/isolcpus-is-deprecated-kinda.html
          - [x] Think how to setup isolated setups so that they fit the non-isolated ones
            - [x] Implement
        - [x] Test isolcpu+pinning
    - [x] First look at Parsec

## Sat 30/Jan/2021

- Studies
  - Linux
    - [x] Check sudo behavior, and ask

- Blog articles
  - [x] Minor updates/fixes to some articles

- QEMU research paper
  - Scripting: Full environment preparation
    - [x] Add projects compiling/copy
    - [x] Split generated CSV files
    - [x] Test/clean/fix `prepare_components.sh`
    - [x] Set the `performance` frequency governor
    - [x] Test/clean/fix `benchmark_pigz.sh`
    - [x] Split diagram creation into separate script (see stash)
  - [ ] Test on bare Ubuntu Server
    - [x] Preparation

## Fri 29/Jan/2021

- Admin/tools
  - O/S
    - [x] Clonezilla: Workaround USB port issue

- Projects/Open source
  - QEMU research paper
    - Scripts: Full environment preparation
      - [x] Assume a standard busybear running locally
      - [x] New directories layout
        - [x] QEMU: can move firmware to a separate dir (`-bios`)?
      - [x] Split projects download/build into a separate script
      - [x] Run QEMU from the benchmark script
        - [x] Fight find snippet for caching sudo permissions
      - [x] Busybear: create empty disk with plenty of space (or extend existing)
      - [x] Linux repo: git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
      - [x] Add projects download and patch
      - [ ] Add projects compiling/copy
        - [x] All except pigz/RISC-V toolchain

## Thu 28/Jan/2021

- Studies
  - Kernel (development)
    - [x] Read paper [Choosing the right timer interrupt frequency on Linux](http://repository.gunadarma.ac.id/313/1/Choosing%20The%20Right%20Timer_UG.pdf)
    - [x] Review all make config options
    - [x] Detail diff with 100Hz, and recompile

- Projects/Open source
  - QEMU research paper
    - [x] Understand details of the interrupt frequency, and update the environment

- Admin/tools
  - Scripting
    - [x] Brogramming: Automated handling of personal notes HEAD commit
  - O/S
    - [x] Prepare a clean ubuntu-server local to an USB key
      - [x] Fight Clonezilla breakages all over

## Wed 27/Jan/2021

- Projects/Open source
  - Busybear/QEMU
    - [x] Open issue with details about improved/working network configuration

- Admin/tools
  - [x] Build new workstation

## Tue 26/Jan/2021

- Studies
  - Busybear/QEMU
    - [x] Finally produce a working network configuration

- QEMU research paper
  - [x] Review providers
    - [x] Verify AWS models and availability
    - [x] Search all bare metal providers
  - [x] Review technical infos for custom kernels/params
  - [x] Prepare/test boot on Ubuntu server
  - [x] Reproduce setup
    - [x] Fix incompatibility with AMD (turned out: OpenSBI bug)
    - [x] Fix issue with emulated smp > 8 not allowed (turned out: kernel used was patched)
    - [ ] Prepare separate environment to use as host
      - [x] Install

## Mon 25/Jan/2021

- Studies
  - Rust
    - [x] Review 3 StackOverflow questions
      - [x] Answer 1
  - GNU Screen
    - [x] First review/test of commands sending
    - [ ] Think strategy for sending commands and waiting
      - [x] Check `tail -f` (source) for inotify

- Admin/tools
  - [x] Planning script
    - [x] Add support for next occurrence period (replan X in Y)
    - [x] Add --compare option
    - [x] Add support for "fixed" time (replan f X)

- Projects/Open source
  - [x] QEMU-pinning
    - [x] Revamp README examples/structure
    - [x] Review and integrate pinning patch
  - QEMU research paper
    - [x] Write test scripts: base version (SSH)
      - [x] End-to-end performance script test
      - [x] Add status messages

## Sun 24/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [ ] AWS Metal
      - [x] Review instances
    - [ ] Write test scripts
      - [x] Bug: `results.data` has int time values

- Studies
  - Gnuplot
    - [x] Simplify project logic, by using stats
    - [x] Improve X tics
    - [x] Minor cleanups
    - [x] Investigate possible bug

## Sat 23/Jan/2021

- Admin/tools
  - [x] Fight issue with BIOS upgrade

- Projects/Open source
  - QEMU research paper
    - [ ] Plan AWS performance testing

## Fri 22/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Execute improved tests
      - [ ] Write test scripts
        - [x] Fixes, base test, minor improvements
        - [x] Review standard deviation
        - [x] Improve directory layout

- Studies
  - Gnuplot
    - [ ] Fight gnuplot tics

## Thu 21/Jan/2021

- Projects/Open source
  - Investigate Busybear issues
    - [x] Test again
      - [x] check start-qemu.sh
      - [x] dump config
      - [x] fight bridge interface
      - [x] fight SSH connection
    - [x] Open other PR
  - QEMU research paper
    - [ ] Execute improved tests
      - [ ] Write test scripts
        - [x] Integrate password in cmdline
        - [x] Minor refactoring
        - [ ] Test/fixes

- Studies
  - Rust
    - [x] Join (part) Rust meetup

## Wed 20/Jan/2021

- Studies
  - Other
    - Gnuplot
      - [x] Review and reorganize existing notes
      - [x] Brutal fight find clean way to print multiple lines from research dataset

- Projects/Open source
  - QEMU research paper
    - [ ] Execute improved tests
      - [ ] Write test scripts
        - [x] Fight configure SSH on Busybear
          - [x] Open Issue+PR
        - [x] Use gnuplot
        - [ ] Test

## Tue 19/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [x] Discuss my pigz/ray tracer preliminary test results
    - [ ] Execute improved tests
      - [x] Review cores pinning mapping
        - [x] Fight hwloc/source for skipping bridges
          - `hwloc`; `lstopo --of console --no-io --no-caches`
      - [ ] Write test scripts

- Blog articles
  - [x] Quickly building a custom Linux (Ubuntu) kernel, with modified configuration (kernel timer frequency)

## Mon 18/Jan/2021

- Blog articles
  - [x] Prepare QEMU user emulation
    - [x] Fight missing library
  - [x] Article RISC-V: Improve the tooling options section

- Projects/Open source
  - geet
    - [x] Fixes related to keyword arguments
  - QEMU-pinning
    - [x] Build script: add capstone dev library dependency #29
    - [x] Structure the repo so that build script/patch changes are recorded #26
    - [x] Build script: point directly to binaries #27
    - [x] Build script: add user space emulation support to build #28
  - QEMU research paper
    - [x] Improved tests preparation
      - [x] Fight install kernel with 100 Hz timer
      - [x] Test RISC-V QEMU with 16 cores

## Sun 17/Jan/2021

- Admin/tools
  - [x] Run interactive+background firefox on a headless machine
    - [x] Fight include missing library

- Studies
  - Rust
    - Articles/Smaller topics/Answers
      - [x] Review answers: 1
    - Ray tracer challenge
      - [ ] Further cleanups
        - [x] Evaluate parallel performance
          - 86% of theoretical max on 8 threads; SMT performance tanks
        - [x] Implement Slab/vector for bidirectional trees
          - Abandoned: very pervasive and time-consuming change
          - For the purpose of optimizing parallelization, the current version is already reasonably efficient
        - [x] Significant refactoring of the ObjParser

- Projects/Open source
  - QEMU research paper
    - [ ] Perfectly parallel test cases
      - [x] prepare ray tracer
        - [x] Prepare project
          - [x] Research RISC-V build
          - [x] Extract sdl interface to separate crate, in order to remove sdl dependency from the library
          - [x] headless_astronaut_rendering: Add commandline arguments
          - [x] headless_astronaut_rendering: Add conveniences for RISC-V build
      - [x] Further scripts tweaking and preliminary testing
        - [x] Prepare base scripts and smoke test
        - [x] headless_astronaut_rendering: Specify number of threads
        - [x] prepare image with ready files
      - [x] Run first pass guest tests
        - [x] run both tests
        - [x] verify `pig` host scalability on the same data
  - QEMU-pinning
    - [x] Add capstone requirement
    - [x] fix bug in vcpu computation and error message

## Sat 16/Jan/2021

- Projects/Open source
  - ZFS installer
    - [Space reclamation issue](https://github.com/saveriomiroddi/zfs-installer/issues/157)
      - [x] Try to reproduce it
      - [x] Think and implement workaround
      - [x] Write wiki page
    - [x] Investigate odd name resolution issue inside jail
      - Cause: VPN provider on the host

- Studies
  - Rust
    - Articles/Smaller topics/Answers
      - [x] Review a few StackOverflow answers

## Fri 15/Jan/2021

- Studies
  - Rust
    - Articles/Smaller topics/Answers
      - [x] Review a few StackOverflow answers
        - [x] Test and reply to one

## Thu 14/Jan/2021

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [ ] Chapter 7

## Wed 13/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [x] Test over-limit RISC-V cpus on all systems

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [ ] Chapter 6

## Tue 12/Jan/2021

- Blog admin
  - [x] Fight latest post not updating on Google, and outdated index
    - [x] Rename incorrect post filenames

- Projects/Open source
  - Geet
    - [x] Clean Ruby 2.7 keyword argument warnings
      - [x] Make Ruby 2.6- compatible
    - [x] Add ruby 3.0 to build
  - QEMU research paper
    - [x] Review scripts
    - [ ] Reproduce paper system
      - [ ] Fight/research max limit of QEMU RISC-V cpus

- Studies
  - Rust
    - Articles/Smaller topics
      - [x] https://stackoverflow.com/questions/45116984/the-trait-cannot-be-made-into-an-object
      - [x] https://stackoverflow.com/questions/38215753/how-do-i-implement-copy-and-clone-for-a-type-that-contains-a-string-or-any-type
  - Javascript/WASM
    - Building JavaScript Games
      - [x] Chapter 5: notes

- Admin/tools
  - [ ] Notes: move old ones/write pending
    - [ ] text_processing.odt (2 pag.)

## Mon 11/Jan/2021

- Blog articles
  - [x] Quick RISC-V cross compilation and emulation

- Projects/Open source
  - QEMU research paper
    - pigz
      - [x] Review source code
      - [x] test scaling
    - [x] Reply, including gathered info

## Sun 10/Jan/2021

- Projects/Open source
  - RISC-V cross-compiling of pigz
    - [x] Test run fedora risc-v and compile(d) pigz

- Blog articles
  - [ ] Quick RISC-V cross compilation and emulation

## Sat 09/Jan/2021

- Projects/Open source
  - QEMU-pinning (for research paper)
    - [x] Build script improvements/extensions
      - [x] Add standard safety shopts
      - [x] Split into functions
      - [x] Add option to specify target arch
      - [x] Add --yes
      - [x] Add option to disable GUI modules
      - [x] General cosmetic cleanups
  - RISC-V cross-compiling of pigz
    - [x] Cross-compile
    - [ ] Fight setup riscv-tools

## Fri 08/Jan/2021

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [x] Chapter 4
      - [ ] Chapter 5 (theory)
      - [ ] Chapter 6

## Thu 07/Jan/2021

- Admin/tools
  - [ ] Notes: move old ones/write pending
    - [ ] rails.odt

- Projects/Open source
  - [ ] Follow up research paper
    - [x] Find good candidates for perfect scaling test
    - [x] Investigate supposed mistake

## Wed 06/Jan/2021

- Blog articles
  - [x] Linux: Associating file types to applications

- Studies
  - Rust
    - Articles/Smaller topics
      - [x] https://stackoverflow.com/questions/25339603/how-to-test-for-equality-between-trait-objects
      - [ ] https://stackoverflow.com/questions/48952627/implementing-partialeq-on-a-boxed-trait
    - Larger topics
      - [ ] [Attribute macros](https://github.com/dtolnay/proc-macro-workshop)
        - [ ] Study
          - [ ] `derive(Builder)`
            - [ ] 01-parse
              - [x] Implement
  - Javascript/WASM
    - Building JavaScript Games
      - [ ] Chapter 4

- Projects/Open source
  - [x] Analyze research paper, and reply

- Admin/tools
  - [ ] Notes: move old ones/write pending
    - [ ] text_processing.odt

- Projects/Open source
  - ZFS installer
    - [x] Support-related documentation updates
      - [x] GitHub discussions: Add reference to README
      - [x] Update discussion pinned post

## Tue 05/Jan/2021

- Blog admin
  [x] Manage permissions

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [x] Chapter 1
      - [x] Chapter 2
      - [x] Chapter 3
  - Articles/Smaller topics
    - Rust
      - [x] This week in rust #365
      - [x] This week in rust #366
        - [x] https://medium.com/centrality/why-work-in-blockchain-journey-from-c-to-rust-developer-eddbc9ccdc3d
      - [x] https://doc.rust-lang.org/nomicon/ownership.html
    - SWE
      - [x] [What is move semantics?](https://stackoverflow.com/questions/3106110/what-is-move-semantics)

- Blog articles
  - [ ] Linux: Associating file types to applications

## Mon 04/Jan/2021

- Blog admin
  - Consultancy
    - [x] Group by year + fix shadows and layout
      - [x] Integrate updated page
        - [x] Fight code not integrating
        - [x] Integrate revision
        - Close order
  - [x] Remove article-specific meta/data from bookshelf page

- Projects/Open source
  - openscripts
    - [x] Extend and open source `bedtime`
  - Drunken tomatoes
    - [x] Upgrade Rails
      - [x] 6.0
      - [x] 6.1
      - [x] Remove rubocop stuff
  - PPA Packaging
    - [x] Skip non-release Ruby versions (which cause issues with the release version upload)

- Studies
  - Javascript/WASM
    - [x] Check out content of the candidate JS books
    - [ ] Building JavaScript Games
      - [ ] Chapter 1

- Blog articles
  - [ ] Linux: Associating file types to applications

## Sun 03/Jan/2021

- Admin/tools
  - [x] Journal entry script: automate subjects index processing

- Studies
  - Rust
    - Ray tracer challenge
      - [ ] Further cleanups
        - [x] Apply new clippy version cleanup
        - [x] Try `weight_max()` Rayon optimization
          - Doesn't exist anymore
          - [x] Remove practically unnecessary overloading &Tuple - &Tuple
        - [x] Use atomic data type for shape IDs, instead of mutex+lazy_static

- Blog admin
  - Consultancy
    - [ ] Group by year + fix shadows and layout
      - [x] Discuss specs
      - [x] Reply other devs

- Projects/Open source
  - `fanotify`-based notification system for backups
    - Test implementation
      - [x] Fight basic fanotify implementation
        - [x] Further fix(es) to the manpage example
        - [x] Prepare and submit test case
        - [x] Fight find systematic location of different linux version man pages
          - [x] Correct bug, using 5.8 version
        - [x] Brutal debug fight
          - Traced back to issue with ZFS
  - ZFS installer
    - [x] Review and redirect support request; update, lock, and pin welcome thread
  - Drunken tomatoes
    - [ ] Upgrade Rails
      - [x] 5.2

## Sat 02/Jan/2021

- Admin/tools
  - [x] Find out how to schedule suspend via Systemd

- Studies
  - Rust
    - Tutoring
      - [x] Arrange meeting

- Blog admin
  - Blog consultancy
    - [x] Find/contact developers

- Projects/Open source
  - `fanotify`-based notification system for backups
    - [x] Review `fanotify` resources
      - [x] Review articles
        - [x] inotify: https://www.thegeekstuff.com/2010/04/inotify-c-program-example
          - [x] write and improve example
        - [ ] https://www.linux-magazine.com/Issues/2017/194/Core-Technologies
          - [x] inotify
        - Research: fanotify examples, and recursion support
          - [x] old: https://stackoverflow.com/questions/1835947/how-do-i-program-for-linuxs-new-fanotify-file-system-monitoring-feature
          - [x] uninteresting: https://stackoverflow.com/questions/59015209/fanotify-problem-with-new-flags-introduced-in-kernel-5-1
          - [x] obsolete: https://stackoverflow.com/questions/39290690/watching-a-directory-tree-whitout-inotify
          - [x] https://lwn.net/Articles/716973
          - [x] https://kernelnewbies.org/Linux_5.1#Improved_fanotify_for_better_file_system_monitorization
    - Test implementation
      - [x] https://www.linux-magazine.com/Issues/2017/194/Core-Technologies
      - [x] `man fanotify` has a full example
      - [x] Quick review [fanotify-rs](https://github.com/ZhangLei-cn/fanotify-rs)
      - [ ] Fight basic fanotify implementation
        - The manpage example doesn't work, and has at least one issue

## Fri 01/Jan/2021

- Studies
  - Rust
    - Tutoring
      - [x] Review codementor.io and send first message

- Blog admin
  - [ ] Split bookshelf by year
    - [x] Review Liquid and implement
    - [x] Fight HTML/CSS
