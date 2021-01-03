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
