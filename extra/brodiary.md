## Fri Nov/20

- Extra
  - [~] Move old notes into personal_notes

- Blog
  - [x] Article: Cleaning email subject prefixes via regexes

- Ray tracer challenge
  - [ ] Chapter 9
    - [x] Create Shape trait
      - [x] Fight PartialEq boxed traits
      - [x] Lots of changes around
      - [x] Try implementation seemingly unused saved_ray()

## Thu Nov/19

- Theory
  - Rust
    - [x] Review MaybeUninit
    - [x] Read: https://stackoverflow.com/questions/27791532/how-do-i-create-a-global-mutable-singleton
    - [x] Review static vars (and update/extend notes)

- Extra
  - [~] Move old notes into personal_notes
    - [x] Reorganize archived resources
  - [x] Fix scraper for incomplete data

## Wed Nov/18

- Theory
  - Rust
    - [x] Advanced review modules
    - [x] Read: https://phaazon.net/blog/rust-traits-privacy
    - [x] Ask design question
    - [x] Study: https://dev.to/takaakifuruse/rust-lifetimes-a-high-wall-for-rust-newbies-3ap
      - Simplify and mention
  - CS
    - [x] Ask terminology question

- Extra
  - [x] Check copyright STOV answers
    - `cc-by-sa` -> https://stackoverflow.blog/2009/06/25/attribution-required
  - [x] Fight how to set custom URL for Firefox search bar

## Tue Nov/17

- Ray tracer challenge
  - [x] Add convenience method `Image#from_pixels`
  - [x] Chapter 7
    - [x] Review chapter
  - [x] Chapter 8
    - [x] Theory
    - [x] Implementation
  - [x] Refactoring: convert `PointLight::new` to use tuples
  - [x] Optimize World#is_shadowed(), by stopping at the first obstruction
  - Fight inheritance/private trait methods
    - Read sources
      - [x] https://users.rust-lang.org/t/how-to-implement-inheritance-like-feature-for-rust/31159/13
      - [x] https://stevedonovan.github.io/rust-gentle-intro/object-orientation.html
      - [x] https://users.rust-lang.org/t/suggestion-for-making-rusts-trait-system-more-usable-for-generics/35155/50
    - [x] Write down all the options

## Mon Nov/16

- Open source
  - Geet
    - [x] Fix Merge PR branch auto deletion test suite
    - [x] Implement Milestone close

- Ray tracer challenge
  - [x] Why doesn't the rayon implementation need move?
  - [x] Several cleanups to Chapter 6 implementation
  - [x] Sdl2Interface: exit only on Esc key
  - [x] Implement a generic I/O Interface trait
    - [x] Design the trait
    - [x] Implement the virtual image type
    - [x] Replace the type in ppm_encoder_test
  - [x] Fight lifetimes
  - [ ] Chapter 7
    - [x] Implementation
      - [x] Extend Image trait, and apply minor adjustments to Sdl2Interface.
      - [x] Add VirtualImage, and put it to use in the PpmEncoder test suite
      - [x] Implement Camera#render
      - [x] Implement World::default(), which replaces new()
    - [x] Practice
      - [x] Chapter 7 practice preparation: create, as refactoring of the previous chapter
      - [x] Parallelize Camera#render()
      - [x] Chapter 7 practice
      - [x] Sdl2Interface: Remove invert_y and origin logic from the default constructor

## Sun Nov/15

- Ray tracer challenge
  - [x] Rename (horrible) ExportToPixels to Image
  - [x] Rename Sdl2Interface "center" concept to "origin"
  - [x] Check out `smart-default` crate, implement in Sphere, if it benefits (-> done)
  - [x] Investigate wall size crash; restrict to smallest test case, and ask question

## Sat Nov/14

- Open source
  - PIA manual connection scripts
    - [x] Update/test PIA PR

- Ray tracer challenge
  - [ ] Chapter 7
    - [x] Debug commit 55151c3f04be077de1449c35dc10f0203d5cdaaa (`cargo run` -> image rendering wrong)
      - [x] Two minor refactorings
  - [x] Parallelize chapter 6 practice!!!

## Fri Nov/13

- Open source
  - PIA manual connection scripts
    - [x] Open fix PR: IPv6 check fails on configuration where IPv6 is disabled via kernel cmdline parameter
    - [x] Open fix PR: Inappropriate permissions for the credentials file

- Extra
  - Admin
    - [x] Test VPN provider new configuration
  - Tooling
    - Schedule maintenance script
      - [x] Allow days without done/todo separator
      - [x] Add day archival to `replan`
        - Other minor improvements

## Thu Nov/12

- Blog
  - [x] Reply blog request

- Extra
  - Admin
    - [x] Fight issue with VPN provider

- Ray tracer challenge
  - [ ] Chapter 7
    - [ ] Implementation
      - [x] Add Matrix::view_transform UTs

## Wed Nov/11

- Ray tracer challenge
  - [ ] Chapter 7
    - Refactorings
      - [x] Add Sphere::from API
      - [x] Test suites: Use `crate::**` instead of importing each type
      - [x] Add convenience method Ray::new()
      - [x] Convert World#objects to array of Sphere references
      - [x] Fight issue with Vec<&> that unknowingly should have been Vec<&mut>
        - Study borrowed references
          - [x] https://stackoverflow.com/questions/28158738/cannot-move-out-of-borrowed-content-cannot-move-out-of-behind-a-shared-referen
          - [x] https://stackoverflow.com/questions/38785744/iterating-through-a-vec-within-a-struct-cannot-move-out-of-borrowed-content
      - [x] Fight mistaken reading of the intersection type specification
    - [~] Implementation

## Tue Nov/10

- ZFS
  - [x] Close (develop) issue "c_boot_partition_size too small" #145
  - [x] Review issue with Debian
  - [x] Close (develop) issue "The EFI system partition and the boot pool partition share the same size"
  - [x] Change default boot pool size
  - [x] Release
  - [x] Test Debian 10.6

- Extra
  - GitHub issues
    - [x] Update description of `demonstrate` opened issue
    - [x] Update description of `rust-sdl2` opened issue, add update, and close
  - Rust
    - [x] Check if the `serial_test` crate can be used with `demonstrate`
    - [x] Think a generic solution for testing in isolation `Sphere::new()`
  - Golang/Terraform: [Issue with Java decompression](https://github.com/saveriomiroddi/terraform-provider-archive-dev/issues/1)
    - [x] Solve issue, and write discussion

- Ray tracer challenge
  - [x] Simplify `Sphere#hit()`
    - [x] Make sure that Sphere#intersections() is in order (review context)
    - [x] Review dot product context

## Mon Nov/09

- Ray tracer challenge
  - [x] Chapter 6
    - [x] Implementation
      - [x] Review lighting logic
      - [x] Fight concurrency issue in Sphere's test suite
  - [x] Convert all owned function params to references
  - [ ] Chapter 7
    - [x] Theory
    - [ ] Implementation
      - [x] Design intersections APIs
      - [x] Fight Rust floats sorting

- Theory
  - Rust
    - [x] Study [trait/references](https://stackoverflow.com/questions/28005134/how-do-i-implement-the-add-trait-for-a-reference-to-a-struct)
      - See related question

- Extra
  - Rust
    - [x] Review UT framework: https://crates.io/crates/spectral

## Sun Nov/08

- Ray tracer challenge
  - [x] Chapter 5
    - [x] Theory
      - Study normalization
        - [x] https://docs.godotengine.org/en/2.1/learning/features/math/vector_math.html
        - [x] https://stackoverflow.com/questions/10002918/what-is-the-need-for-normalizing-a-vector
        - [x] https://stackoverflow.com/questions/6875055/raytracing-when-to-normalize-a-vector
      - [x] Review the chapter
    - [x] Implementation
      - [x] Think intersections type(s) design
        - postpone, given too little infos
    - [x] Practice
      - [x] Fight position, sphere size and transformations
      - [x] Add convenience APIs
      - [x] Experimentation
    - [x] Post 2 erratas
  - [ ] Chapter 6
    - [x] Theory
    - [~] Implementation
      - [x] Fight errors caused by wrong test preset value
    - [x] Practice
  - [~] Convert owned function params to references, and remove `clone()` invocations

## Sat Nov/07

- Extra
  - [x] Schedule maintenance script
    - [x] Simplify: Remove previous_date
    - [x] Add support for first date not being the current one

- Ray tracer challenge
  - [x] Chapter 4
  - [x] Chapter 5
    - [~] Theory
      - Stumble on reason for normalization

## Fri Nov/06

- Ray tracer challenge
  - [x] Chapter 3
    - [x] Fight branchless cofactor
    - [x] Implementation
  - [ ] Chapter 4
    - [x] Theory
    - [x] Fight 2d arrays formatting and evaluate alternatives
    - [ ] Implementation
      - [x] Translation

- Extra
  - [x] Fight understand case where the negative lookahead workaround works with a `^` but not `x`
  - [x] Report reader bug

## Thu Nov/05

- Theory
  - Rust
    - [x] [Generics (Java <> Rust)](https://gist.github.com/Kimundi/8391398)
    - [x] https://stackoverflow.com/questions/54465400/why-does-returning-self-in-trait-work-but-returning-optionself-requires

- Ray tracer challenge
  - [ ] Chapter 4
    - [~] Theory
  - [ ] Chapter 3
    - [ ] Implementation
      - [x] Merge Matrix multiple types into a single one, using Vec
      - [x] Add root `describe!` scope, to workaround API's limitation
      - [x] Submatrix
      - [x] Minor

## Wed Nov/04

- Extra
  - [x] VSC: Fight find out [how to disable suggestions but leave snippets completion](https://code.visualstudio.com/docs/editor/intellisense#_tab-completion)

- Ray tracer challenge
  - [ ] Chapter 3
    - [ ] Implementation
      - [x] Search multiple supertraits, and support of generic `Self` type
      - [x] Fight traits/boxed types/Self

## Tue Nov/03

- Extra
  - Ask: comparsion between different macro implementations
  - Write schedule maintenance script
    - [x] Base design
    - [x] First stage: fix dates
    - [x] Second stage
    - [x] Last stage, with restructurings
    - [x] Refactor: make use of `find_preceding_or_existing_date`

- Ray tracer challenge
  - [ ] Chapter 3
    - [ ] Implementation
      - [x] Tuple: Add some convenient functions
      - [x] Implement Matrix-Tuple multiplication
      - [x] Implement identity matrix
      - [x] Implement matrix transposition

## Mon Nov/02

- Articles
  - [x] Research and fix nmon issue with reload (restart)
  - [x] November #1: Chef run as alternate user

- Extra
  - Regexes
    - Ask difference between lookarounds for specific case

- Ray tracer challenge
  - [ ] Chapter 3
    - [ ] Implementation
      - [x] Fight generate structs via macro
      - [x] (Fix issue with existing code)
      - [x] (Further) study uninitialized memory APIs
      - [x] Other epsilon error

## Sun Nov/01

- Ray tracer challenge
  - [ ] Chapter 3
    - [x] Theory

## Fri Oct/30

- Ray tracer challenge
  - [ ] Chapter 3
    - [~] Theory

## Thu Oct/29

- Extra
  - Regexes
    - [x] Help other user with another use case
  - New tablet
    - [x] Update
    - [x] Fight wifi on
      - [x] Sleepywifi
    - [x] Test battery

## Wed Oct/28

- Extra
  - Regexes/Thunderbird
    - [x] Configure addon and regex for compacting multiple email title prefixes
      - [x] Help other user with other use case

- Theory
  - Design
    - [x] https://technology.riotgames.com/news/future-leagues-engine
  - Rust
    - [x] Frequent traits: https://stevedonovan.github.io/rustifications/2018/09/08/common-rust-traits.html

- SDL (Ray tracer challenge/Libemuls)
  - [x] Ask how to redraw the screen [in rust]

- Open source
  - [x] Follow up issue on demonstrate

- Ray tracer challenge
  - [x] Chapter 2
    - [x] Practice exercise
      - [x] Extract HasFloat64Value, and implement, along with overloaded functions, in Tuple
      - [x] Add Color#u8_components, and adjust to match the book's logic
      - [x] Fight correct mistakes due to allowed warnings -> disable warnings

## Tue Oct/27

- Theory
  - Rust
    - [x] [Method overloading](https://stackoverflow.com/q/24857831)
    - [x] Composition
      - https://hackernoon.com/why-im-dropping-rust-fd1c32986c88
        - https://news.ycombinator.com/item?id=12474445
        - https://github.com/rust-unofficial/patterns/blob/master/anti_patterns/deref.md
    - [x] `Read`/`Write` traits and usages

- Blog admin
  - [x] Rename `_books` to `_bookshelf`

- SDL (Ray tracer challenge/Libemuls)
  - [x] Simplify SDL logic, by using the logical size
    - Fight canvas update (was not invoking update() on init)
    - Fight redraw on resize (couldn't find how to redraw when resizing)

- Ray tracer challenge
  - [ ] Chapter 2
    - [x] Theory+exercises
      - [x] Other fight with pixel reading
      - [x] Implement overloaded `Color::new()`!

## Mon Oct/26

- Ray tracer challenge
  - [ ] Chapter 2
    - [~] Theory+exercises
      - [x] Fix mistake with epsilon
      - [x] Fight multiple issues with SDL library
        - extremely slow pixel read, integration with test suite, improper pixel read structure, read pixels not working stably
      - [x] Fight accidental omission of the screen update

## Sun Oct/25

- Theory
  - C++/SWE
    - [x] Read C++ templates: http://people.cs.uchicago.edu/~jacobm/pubs/templates.html
      - [x] constexpr vs. templates metaprogramming: https://stackoverflow.com/questions/59538054/constexpr-vs-template-metaprogramming-performance-differences
      - [x] https://www.modernescpp.com/index.php/c-core-guidelines-programming-at-compile-time-with-constexpr
      - [x] https://www.variadic.xyz/2019/07/03/constexpr2

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [ ] Building a Binary Tree to Efficiently Store and Sort Data
        - [x] Study
        - [x] Think binary trees printing algorithm
          - [x] Review established solutions (favorite: https://stackoverflow.com/a/8948691/210029)

## Sat Oct/24

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [x] Viewing Data in Both Directions with Doubly Linked Lists
        - [x] Implement exercise: pop back
        - [x] Fight: implement iteration without deep clone of the `next` reference

## Fri Oct/23

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [ ] Viewing Data in Both Directions with Doubly Linked Lists
        - [x] Re-study `Rc/RefCell`
          - https://stackoverflow.com/questions/57367092/what-is-the-difference-between-rcrefcellt-and-refcellrct
          - https://github.com/rust-lang/book/issues/1543#issuecomment-696995826
        - [x] Understand and copy to notes the iterative weak reference madness
          - https://stackoverflow.com/questions/36597987/cyclic-reference-of-refcell-borrows-in-traversal
        - [x] Review push_back (7:52) and implement 🤯
        - [x] Implement exercise: pop front
          - [x] Bloody fight with Rc/Weak references and unwrapping
        - [x] Study https://stackoverflow.com/questions/27687872/cannot-move-out-of-dereference-of-mut-pointer-when-calling-optionunwrap

## Thu Oct/22

- Blog
  - [x] Article: Monitoring EC2 CPU Steal time on AWS instances
  - [x] Ruby Tk bindings article: Update to Ubuntu 20.04

- Emulation
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Implement Returns
        - [x] RET
        - [x] RET cc, nn
    - [x] Cleanup and merge current state
    - [ ] Implementation: basic CPU
      - [ ] Remaining instructions
        - [ ] Extend test framework to accommodate interrupts testing (*expected*)
          - [x] Design

## Wed Oct/21

- Blog
  - [~] Article: Monitoring EC2 CPU Steal time on AWS instances

- Ray tracer challenge
  - [x] Chapter 1
    - [x] Complete library exercises
      - [x] Correct the exercises to use tuples/vectors/points according to the test specs!
    - [x] Split in crates (`library`, `practice`)
    - [x] Practice exercise
    - [x] Think about separating Vector/Point types
      - [x] Review Subclassing in Rust
        - [x] Deprecated trait objects
      - [x] Make conclusion

## Tue Oct/20

- Ray tracer challenge
  - [ ] Chapter 1
    - [x] Complete theory

- Emulation
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Implement Calls
        - [x] Add new category of testing for conditional jumps
          - [x] Simplify previous tests
          - Adding the conditional jumps support was expected, however not this little tricky part
        - [x] CALL cc, nn
      - [x] Implement Restarts
        - [x] RST n
          - [x] Handle address not encoded as parameter, but in the opcode (*unexpected*)

## Mon Oct/19

- Ray tracer challenge
  - [ ] Chapter 1
    - [x] First couple of exercises
      - [x] Try cucumber
      - [x] Search crate with assertions for floats

- Extra
  - Tooling
    - git: configure aliases for copying commit titles/messages at relative HEAD positions
  - Community
    - [x] Add `cucumber-rust` to Awesome Rust
    - [x] Report/fix `cubumber-rust` improper badge

## Sun Oct/18

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Discussion about feasibility of non-ROM-based automated testing
      - [x] Implement Rotates & Shifts
        - All: manual `cf`; no `hf`
        - [x] RLCA
        - [x] RLA
          - [x] Tests generator: Order flag expectations (*unexpected*)
            - The automation in the test generation is proving confusing, although this was a simple addition.
        - [x] RRCA
        - [x] RRA
        - [x] RLC r
          - [x] Open upstream fix PR
          - [x] Add RLCA data transform, and regenerate metadata
        - [x] RLC (HL)
        - [x] RL r
          - [x] Check if there is the same issue of RLCA and fix
            - [x] Scan opcodes: other three found
              - [x] Fix
            - [x] Update Issue/PR
            - [x] Regenerate data + fix UTs
          - [x] Brief check documentation (gekkio); test Latex editors
        - [x] RL (HL)
        - [x] RRC r
        - [X] RRC (HL)
        - [x] RR r
        - [x] RR (HL)
        - [x] SLA r
        - [x] SLA (HL)
        - [x] SRA r
        - [x] SRA (HL)
        - [x] SRL r
        - [x] SRL (HL)
      - [x] Implement Bit Opcodes
        - [x] BIT n, r
        - [x] BIT n, (HL)
        - [x] SET n, r
        - [x] SET n, (HL)
        - [x] RES n, r
        - [x] RES n, (HL)
      - [x] Implement Jumps
        - [x] Add support for manually specifying the PC expectation (*expected*)
        - [x] Add support for `cc` condition (*expected*)
        - [x] JP nn
        - [x] JP cc, nn
        - [x] JP (HL)
        - [x] JR n
        - [x] JR cc, n
      - [ ] Implement Calls
        - [x] CALL nn

- Blog
  - [x] Discuss tcl/tk libraries version with reader

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [~] Viewing Data in Both Directions with Doubly Linked Lists
        - [x] Implement `push_front()`
        - [x] Massive fight against iterating the `Rc/RefCell` references

## Sat Oct/17

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [x] Creating a Linked List
        - [x] Fight borrow checker: found solution
        - [x] Exercise

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Implement Miscellaneous
        - [x] SWAP (HL)
        - [x] DAA
        - [x] CPL
        - [x] CCF
        - [x] SCF

- Theory
  - Rust
    - [~] Understand BCK issue with linked list iterative traversal
      - [x] https://stackoverflow.com/questions/50251487/what-are-non-lexical-lifetimes
        - non-lexical lifetimes
          - [x] https://stackoverflow.com/questions/32300132/why-cant-i-store-a-value-and-a-reference-to-that-value-in-the-same-struct
          - [x] https://stackoverflow.com/questions/38023871/returning-a-reference-from-a-hashmap-or-vec-causes-a-borrow-to-last-beyond-the-s
          - [x] https://stackoverflow.com/questions/32761524/why-does-hashmapget-mut-take-ownership-of-the-map-for-the-rest-of-the-scope
          - [x] https://stackoverflow.com/questions/41187296/cannot-borrow-as-immutable-because-it-is-also-borrowed-as-mutable-in-function-ar
          - [x] https://stackoverflow.com/questions/47395171/how-to-update-or-insert-on-a-vec
          - [x] https://stackoverflow.com/questions/41601197/is-there-a-way-to-release-a-binding-before-it-goes-out-of-scope
            - [x] https://stackoverflow.com/questions/30243606/why-is-a-borrow-still-held-in-the-else-block-of-an-if-let
              - [x] https://bluss.github.io/rust/fun/2015/10/11/stuff-the-identity-function-does/
          - [x] https://stackoverflow.com/questions/37986640/cannot-obtain-a-mutable-reference-when-iterating-a-recursive-structure-cannot-b

## Fri Oct/16

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [ ] Creating a Linked List
        - [x] Fight borrow checker, day 2
          - [x] Ask on forum
      - [~] Viewing Data in Both Directions with Doubly Linked Lists

## Thu Oct/15

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Implement 8-bit ALU
        - [x] SBC A, n
          - Add to metadata (it's missing in the reference document)
        - [x] AND A, r
          - [x] Correct missing false/true case in h/c flags code generation
        - [x] AND A, (HL)
        - [x] AND A, n
        - [x] OR A, r
        - [x] OR A, (HL)
        - [x] OR A, n
        - [x] XOR A, r
        - [x] XOR A, (HL)
        - [x] XOR A, n
        - [x] CP A, r
          - [x] Fix missing carry position data in `CP` instructions
        - [x] CP A, (HL)
        - [x] CP A, n
        - [x] DEC r
        - [x] DEC (HL)
      - [x] Implement 16-bit arithmetic
        - [x] ADD HL, rr
        - [x] ADD SP, n
        - [x] INC rr
        - [x] DEC rr
      - [x] Add support for prefixed instructions
        - [x] Convert metadata prefix format (*improvable*)
          - Could have thought better
      - [ ] Implement Miscellaneous
        - [x] SWAP r

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [ ] Creating a Linked List
        - [~] Exercise
        - [~] Fight borrow checker

## Wed Oct/14

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 3
      - [~] Creating a Linked List
      - [~] Viewing Data in Both Directions with Doubly Linked Lists

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 8-bit ALU
        - [x] Tests generator: Improve tests skipping (*unexpected*)
          - Testing SBC A, A got difficult with the existing logic
          - Update existing tests
        - [x] SBC A, r
        - [x] SBC A, (HL)
          - Fight H flag semantic mistake in test

## Tue Oct/13

- [x] Add `strum` to notes

- [~] Terraform: issue raised about Java
  - [x] write a test executor

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Significant testing APIs extension and simplification
        - Fight design
        - Sad drop of implicit register expectations
      - [x] Implement forgotten: POP AF

## Mon Oct/12

- Study macros: https://danielkeep.github.io/quick-intro-to-macros.html
  - [x] hygiene

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [~] Testing: Extend to all registers, include `[A]F`, and simplify
        - The refactoring has been driven by the required addition of the `[A]F` registers support, which was causing way too much logic to be duplicated
        - Additional logic is required due to handling of flag register <> individual flag by the macro (*unexpected*)
        - [x] Refactor: use cycles (required new crate) to test registers
        - [x] Add remaining registers, including `[A]F`
      - [¬] Implement forgotten: POP AF

## Sun Oct/11

- [x] Extra: Test Ripcord

- Training/CS+Rust
  - Hands-On Data Structures and Algorithms in Rust
    - [x] Setup repository
    - [x] Fix source material st00pid warnings
    - [x] Separate implementation into modules
    - [x] Write macro for unit testing (and add UTs)
      - Fight variable scoping
      - Fight module scoping
    - [x] Improve merge sorting, and clear the old implementations
    - [x] Section 2
      - [x] Quicksort
      - [x] Quicksort with pseudo random
      - [x] Threaded quicksort
      - [x] Rayon/work stealing
      - [x] Dynamic programming
      - [x] Test

## Fri Oct/09

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Remove unused (and broken) functionality to set custom PC
        - may be required, if so, will add in correct form
      - [ ] Implement 8-bit ALU
        - [x] ADD A, r
          - [x] Correct flag automatic generation (in execution)
        - [x] ADD A, (HL)
        - [x] ADD A, n
        - [x] ADC A, r
        - [x] ADC A, (HL)
        - [x] ADC A, n
        - [x] SUB A, r
        - [x] SUB A, (HL)
        - [x] SUB A, n

## Thu Oct/08

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Implement 16-bit loads
        - [x] PUSH rr
        - [x] POP rr
      - [ ] Implement 8-bit ALU
        - [~] ADD A, r
          - Fight bug with carry on the 8th bit with 8-bit operands

## Wed Oct/07

- Extra
  - [x] Test workaround for VSC viewport jumping issue (https://git.io/JUxB1)
  - [x] Report

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Further review algorithmic decoding

## Tue Oct/06

- ZFS
  - [x] Test and release keyboard layout issue

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Correct half/carry implementation (*unexpected*)
        - [x] Add carry metadata to `opcodes_data_integration.rb`
          - [x] Add `instructions.json` carry metadata integration
        - [x] Correct existing implementations
        - [x] Complete automatic flags code generation
      - [x] Move immediates decoding from the execution to the decoding
      - [ ] Registers algorithmic decoding possible implementation (*unexpected*)
        - [x] Review
        - [x] Think of generator/cpu architectural differences
      - [x] Allow tests skipping: lost commit!?
      - [ ] Implement 16-bit loads
        - [x] `LDHL SP, n`

## Mon Oct/05

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement signed/carry operations
        - [x] Make 100% sure that `c` flag can be bit 8 or 16, depending on context (review emulators)
          - !!! Different for each case !!!
          - `ADD HL, BC` (0x09)
            - `SameBoy/Core/sm83_cpu.c:470`
            - `visualboyadvance-m/src/gb/gbCodes.h:1235`
        - [x] Study exactly `carry` (out/in) and `overflow` flags (and `borrow`)
          - [x] carry/overflow: http://teaching.idallen.com/dat2343/10f/notes/040_overflow.txt
            - overflow: sign changed in signed operation
          - [x] carry formula: https://stackoverflow.com/q/62006764 + https://stackoverflow.com/q/20494087
          - [x] carry in<>out: https://www.quora.com/What-does-carry-in-and-carry-out-mean-in-electronics
            - in/out: just the wiring of the ports
        - [x] Review half/carry implementation in common GB emulator
        - [x] Remove dead method set_flags(), and also not needed `compose_address()`
        - [x] Convert flags to register (*unexpected*)
          - [x] Fight flag/lifetimes in Index
        - [ ] Correct half/carry implementation (*unexpected*)
          - [x] Automate half/carry implementation
            - [x] Flags: change definition: `null`, `true`, `false`, `<n>` -> position for carry
          - [x] `INC r`
          - [~] Correct existing implementations

## Sun Oct/04

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 16-bit loads
        - [x] Review half/carry implementation in common GB emulator
      - [x] Fix new testing issues
      - [x] Testing framework improvements
        - [x] Allow multiple tests per testing flag type
        - [x] Allow tests skipping (*unexpected*)

## Sat Oct/03

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 16-bit loads
        - [x] Support more than two registers for an operation in the implementation (*difficult*)
          - [x] Repair templates generator
            - [~] Use overlapping unions for registers
              - [x] Review macros to declare the 8 bit field names, e.g. `ah`/`al`,  instead of `h`, `l`
                - See https://is.gd/r1Fa8A
            - [x] Complete repair
              - [x] Per-opcode generation doesn't work anymore
              - [x] When running in per-opcode mode, don't print the `WRITEME`
            - [x] Some renames, to improve readability
        - [~] Implement `LDHL SP, n` (77)
          - [~] Find out exact semantics of `H` flag, and dependency on `N`
        - [x] Implement `LD (nn), SP`
          - [x] Add support for testing an array of memory to the testing APIs (*difficult*)
            - Fight macro/passing array reference

## Fri Oct/02

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 16-bit loads
        - [ ] Support more than two registers for an operation in the implementation (*difficult*)
          - [x] Instructions data: Move the operand types to the instruction level (*mistake*)
          - [x] Simplify instructions with multiple flag outcomes (*difficult*)
            - [x] Instructions data: Correct `SUB A, R` (and similar) (*unexpected*)
            - [x] Move flags data to the instruction level
          - [~] Repair templates generator

## Thu Oct/01

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 16-bit loads
        - [ ] Support more than two registers for an operation in the implementation (*difficult*)
          - [x] Add the operand type to the instructions data (*unexpected*)
          - [x] Move docs to their own directory

## Wed Sep/30

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 16-bit loads
        - [ ] Support more than two registers for an operation in the implementation (*difficult*)
          - [x] New implementation with enums
          - [x] Add instructions data generator (breaks templates generator)
          - [x] Pseudo-Manually copy all the instruction data into metadata
          - [x] Remove implied registers from the JSON

## Tue Sep/29

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 16-bit loads
        - [ ] Support more than two registers for an operation in the implementation (*difficult*)
          - [x] Review structure with separate scopes; reply question
          - [x] Review [`Index`/`IndexMut`](https://is.gd/bGBIJH)
            - [x] Review: How does it workaround the borrowing rules?
            - [x] Copy `Index[Mut]` to notes
          - [~] New implementation with enums
            - [x] Inspect the number and nature of register

## Mon Sep/28

- ZFS
  - [x] Review PR
  - [x] Update Shellcheck link (the current started breaking the build)

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement 16-bit loads
        - [ ] Implement `LDHL SP, n` (77)
          - [x] Handle signed immediate
          - [x] Think general problem of `H` flag
          - [x] Fight general problem of `uM + iN # M > n`
            - [x] Think neat solution
        - [ ] Support more than two registers for an operation in the implementation (*difficult*)
          - [x] Ask question about method design
          - [x] Copy all the commands
          - [x] Review alternatives to to current method design

## Sun Sep/27

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Simplify `FF00 + C` logic (can just compute inside the exec block!)
        - [x] Add `indirect` attribute to JSON
      - [x] Split generator into multiple files (*unexpected*)
      - [x] Implement `LD SP, HL`
      - [~] Implement `LDHL SP, n`

## Sat Sep/26

- Emulation/Rust
  - Game Boy
    - Review: https://blog.ryanlevick.com/DMG-01/public/book/introduction.html
    - [ ] Implementation: basic CPU
      - [x] Add LDD/LDI
      - [x] component_sharp_lr35902: generator: Simplify logic via new, richer, class OperandType
      - [ ] Simplify `FF00 + C` logic (can just compute inside the exec block!)
        - [x] Basic implementation

## Wed Sep/23

- Extra
  - ZFS
    - [x] Discuss issue

## Tue Sep/22

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Implement architectural change for LDD
      - [x] Refactoring: Merge `operation_code` with `flags_code`? that's what testing does
      - [x] Massive nightmare: Improve (now required) solution for operations involving twice the same register
        - [x] Requires: push down the address calculation
        - [x] Study Rust unsafe code
        - [x] Good time to: Improve naming for operands/values
      - [x] Move `extra` components to the respective component crates
      - [x] Implement `LHD A, (n)`
      - [x] component_sharp_lr35902: generator: Generalize the operand type-based switch/case (*unexpected*)
      - [x] Implement `LD r, nn`
        - [x] Handle `SP` registers being split in two when passed (*unexpected*)

## Sat Sep/19

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Study (overlapping) unions - wow!!
        - [x] understand why Copy+Clone are required

## Fri Sep/18

- Extra
  - Passenger
    - Allow sections skipping in `passenger-mem-stats`
      - Fight odd newlines introduced
  - Minor discussions/searches

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] LD A, FF00 + C
      - [x] LD FF00 + C, A
      - [ ] Think architectural change for LDD

## Thu Sep/17

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Review the manual for the correctess of the current metadata/logic
      - [x] Implement instructions: 100% coverage of current metadata
        - [x] Fix failures
          - [x] Remove the `pre` concept from the testing API
          - [x] Allow expressions for `mem` expectations
        - [x] Fixup commits

## Wed Sep/16

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement instructions: 100% coverage of current metadata
        - [x] Fix a couple of bugs
        - [x] Add flags testing support
        - [x] Improve naming, minor cleanups, and update document
        - [x] In tests, add round brackets if indirect

- Extra
  - Fight setup new printer

## Tue Sep/15

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Implement instructions (via script extension): base version
        - [x] Add test metadata fragments support
        - [x] Large refactoring
      - [ ] Implement instructions (via script extension)
        - [x] Fill some spec data
        - [x] Simplify tests structure and improve flags testing
        - [ ] Add flags testing support

- Extra
  - [x] Archive past resources; update current/pool tasks
  - [x] Search other Ruby editor, out of VSC Ruby desperation

## Tue Sep/15

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement instructions (via script extension): base version
        - [x] Implement UT templates
        - [x] Add memory testing support to assertion
        - [x] Allow replacing the templates multiple times

## Mon Sep/14

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Implement instructions (via script extension)
        - [x] Implement `LD nn, n`
        - [x] Implement `LD r1, r2`
        - [x] Implement `LD A,n`
          - convert `HL_location()` to `indirect_reference(register)`
        - [x] Reimplement `NOP` as table-based
        - [x] Implement automatic insertion in `cpu.rs`, and cargo check
        - [x] Fix existing issues
          - (follow up)
          - [x] Big borrow problem
            - [x] pass all the flags as mutable
            - [x] convert `indirect_address()` to `compose_address()`
            - [x] add `REGISTER_INDIRECT` operand_type
            - [x] merge `execute_LD A, nn` into `LD r, nn`
          - [x] Remaining changes to make the output valid
          - [x] Update document

## Sat Sep/12

- Emulation/Rust
  - [x] Minor cleanups
    - [x] Rename `*-interfaces` to `interfaces-*`
    - [x] Remove distributed systems mention from README
    - [x] Switch testing framework to `demonstrate`
      - [x] Report issue with mutable variables in `before` block
    - [x] Fight complete `assert_cpu_execute!` macro
      - [x] Ask for cleaner way of writing it
      - [x] Evaluate a mixed approach macro + struct+default()
    - [ ] Implementation: basic CPU
      - [x] Write script base for converting a few https://gbdev.io/gb-opcodes/Opcodes.json to UTs and Code
        - [x] Update generated testing code with new macro, and add automated test values
        - [x] Follow up with generator base structure
        - [x] Follow up with first instruction: `INC n`, without UTs

- Extra
  - [x] Investigate and report unnecessary warning on ruspec/demonstrate

## Wed Sep/09

- ZFS
  - [x] Review  mount issue (deadline: 09/Sep)
    - Reference: https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2018.04%20Root%20on%20ZFS.html
      - 4.8b: /boot/efi
      - 5.7: /boot

## Tue Sep/08

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [x] Rename flag variables
      - [ ] Write script base for converting a few https://gbdev.io/gb-opcodes/Opcodes.json to UTs and Code
        - [x] Review CPU sources; https://rednex.github.io/rgbds/gbz80.7.html is unhelpful, as it misses the opcodes
        - [x] Review opcodes json #1, and start separating the concepts that can be translated
        - [x] Join discord, and ask the `immediate` meaning (seems the opposite of what it is)
        - [x] Raise issue with maintainers about `immediate` misnaming (see https://bit.ly/33boc9l)
        - [x] Implement cycle counts (`bytes`)
        - [x] Implement flags (`flags`)
        - [x] Review opcodes json #2 for further concepts translation
        - [ ] Follow up implementation execution generator
          - [x] Inspect INC n
          - [x] Inspect LD nn, n
          - [ ] Write macro for test fx invocation (update personal notes)
            - [x] Study macros
            - [x] Fight find implementation of optional parameters along with the required style
            - [ ] Implement

## Mon Sep/07

- Emulation/Rust
  - Game Boy
    - [ ] Implementation: basic CPU
      - [ ] Write script base for converting a few https://gbdev.io/gb-opcodes/Opcodes.json to UTs and Code

- Extra
  - [x] Understand proper testing descriptions
    - [x] Review and copy examples
  - [ ] Review: https://simpleprogrammer.com/ultimate-list-developer-podcasts
  - [x] Pandocs project: test local run, to research the pdf export

## Sat Sep/05

- Extra
  - [ ] Understand proper testing descriptions
    - [x] Quick look at the RSpec book

## Fri Sep/04

- Emulation/Rust
  - [x] Extra
    - [x] Find out how to print Pan Docs in acceptable PDF
    - [x] Add Demonstrate to Awesome-Rust
    - [x] Review Demonstrate as test framework
  - Game Boy
    - [ ] Theory
      - [x] Study [Emulation of Nintendo Game Boy](https://git.io/JUtp6)

## Thu Sep/03

- ZFS
  - [x] Review issue/reply

- Emulation/Rust
  - Scheduler
    - [x] Locking Readings
      - Spinlocks/Mutexes: https://matklad.github.io/2020/01/02/spinlocks-considered-harmful.html
      - Linux reply: https://www.realworldtech.com/forum/?threadid=189711&curpostid=189723
      - Reference: https://en.cppreference.com/w/cpp/atomic/memory_order
      - Explanation: https://stackoverflow.com/questions/12346487/what-do-each-memory-order-mean
      - Explanation, great!: https://gcc.gnu.org/wiki/Atomic/GCCMM/AtomicSync
  - Game Boy
    - [ ] Theory
      - [x] Think basic design/components interrelation
      - [x] Review all available guides
    - [ ] Implementation: basic CPU
      - [x] Write base struct + UT
      - [x] Write one decode + UT

## Wed Sep/02

- Extra
  - [x] Switch to alternative Android sync service, after Dropbox disaster

## Tue Sep/01

- Emulation/Rust
  - Scheduler
    - [x] Implementation of a fast enough scheduler (insane)
    - [ ] Readings
      - [ ] https://matklad.github.io/2020/01/02/spinlocks-considered-harmful.html

## Mon Aug/31

- Emulation/Rust
  - Scheduler
    - [ ] Think scheduler
    - [x] Playground: improve, reorganize, and review
    - [x] Experiments with async
    - [x] Cleanup the async experiments, merge, and conclude
  - [ ] Theory
    - [ ] Study [Emulation of Nintendo Game Boy](https://git.io/JUtp6)
  - Misc (Project)
    - [x] Review and merge playground
    - [x] Rename the CHIP-8 component to system

## Sun Aug/30

- Extra
  - [x] Add missing testing resources to awesome_rust
    - [x] Fix PR
  - [x] Move `brodiary.md` to `personal_notes`
  - [x] Discuss issue with rust-analyzer

- Emulation/Misc
  - [x] Discord
    - [x] Fight setup discord
    - [x] Ask question about idling
    - [x] Threading discussion
  - [x] Investigate Atari 2600 emu documentation/find alternative
    - Alternatives
      - Space invaders, Pacman, Gameboy, GBC, Sega master system and Sega Genesis
      - Apple-II
      - NES
      - GB/NES
      - SMS
  - [x] Reorganize resources

- Emulation/Rust
  - [x] Inbetween tasks/cleanups
    - [x] Improve naming
      - [x] Think names
      - [x] Rename
      - [x] Update `README.md`
    - [x] Integrate playground!
    - [x] Move CHIP-8 references to wiki
    - [x] All libraries: uniform error handling: either `unwrap()` or return `Result<(), Box<dyn Error>>`
    - [x] Fight rust-analyzer nested project
      - [x] participate discussion: https://github.com/rust-analyzer/rust-analyzer/issues/3265
  - [x] Scheduler/Concurrency theory
    - [x] Decide what next #1
    - [x] Investigate idling without sleep
      - concepts: HLT, HPET, TSC
      - pause concept
        - https://stackoverflow.com/questions/12894078/what-is-the-purpose-of-the-pause-instruction-in-x86
        - https://stackoverflow.com/questions/7086220/what-does-rep-nop-mean-in-x86-assembly-is-it-the-same-as-the-pause-instru
    - [x] Experiment further with sleep/sync implementation
      - Conclusion: Atomic data types are very fast; follow up on building the machine, and think about communication when it's time.
  - Game Boy
    - [ ] Theory
      - [x] Review what to study, and dump to offline format
      - [ ] Study [Emulation of Nintendo Game Boy](https://git.io/JUtp6)

## Sat Aug/29

- Emulation/Rust
  - [ ] Scheduler/Concurrency theory
    - [x] Research designs/performance #1
    - [x] Ask how to trivially write filler operations
    - [x] Find concurrency resources
    - [ ] Reads
      - [x] https://rust-lang.github.io/async-book

## Fri Aug/28

- Emulation/Rust
  - Emulators design/concurrency theory
    - [x] https://arstechnica.com/gaming/2011/08/accuracy-takes-power-one-mans-3ghz-quest-to-build-a-perfect-snes-emulator/
    - [x] https://byuu.net/design/schedulers/
    - [x] https://forums.nesdev.com/viewtopic.php?f=3&t=8999
    - [x] https://news.ycombinator.com/item?id=3987660
  - [ ] Scheduler/Concurrency theory
    - [x] Research designs/performance #1
    - [x] Interesting APIs (take note): Barrier/Condvar
    - [ ] Reads
      - [x] https://blogs.msmvps.com/peterritchie/2007/04/26/thread-sleep-is-a-sign-of-a-poorly-designed-program
      - [x] Review sleep in higan scheduler
  - [ ] Atari 2600
    - [ ] Study "Making Games for the Atari 2600"
      - [ ] Few chapters
  - [ ] Scheduler/Concurrency theory
    - [ ] Test async approach
      - [ ] Implement async simple scheduler
        - [x] First attempt, with sleep

- Extra
  - Method chaining challenge from colleague

## Thu Aug/27

- Emulation/Rust
  - Inbetween tasks/cleanups
    - [x] Plan next
      - [x] Machine (or WASM) -> Atari 2600
      - [x] Books -> Making Games for the Atari 2600
    - [x] Check if can contribute to https://github.com/tobiasvl/awesome-chip-8
  - [ ] Atari 2600
    - [ ] Study "Making Games for the Atari 2600"
      - [ ] Chapter 01

- Extra
  - [ ] Add missing testing resources to awesome_rust

- Rust/study
  - [x] Study testing chapter in Rust reference
    - [x] Fight find testing frameworks, and review (-> ruspec)
  - [ ] Find how to get Rust review
    - [x] General find
      - https://www.quora.com/Where-can-I-find-experts-to-code-review-my-GitHub-code

- ZFS
  - [x] Review and reply encryption feature request

## Wed Aug/26

- Emulation/Rust
  - [x] CHIP-8
    - [x] Add documentation where missing
    - [x] write README
    - [x] release

- Blog
  - Test Inkscape for diagrams

- Extra
  - Visual Studio Code launch configuration
    - Make Rust (extensions) handle a project inside another project
  - Update/structure personal notes about sdl2/rust sound
  - Follow up RSpec issue investigation
  - Forward WxWidgets technical explanation

## Tue Aug/25

- Emulation/Rust
  - CHIP-8
    - [x] Sound interface
      - [x] Rip out previous (misunderstood) beep logic
      - [x] Debug audio timer
      - [x] Improve max speed logic
      - [x] Test
    - [x] Review overall interfaces structure/organization
      - [x] Review
      - [x] Merge commits
      - [x] Rename DEVICE_FREQUENCY to AUDIO_DEVICE_FREQUENCY
      - [x] Review branch history warnings/errors
      - [x] Split `interfaces` into modules: `audio`, `video`, `logging`, `events`
      - [x] Review imports (curly braces etc.)

- Extra
  - Visual Studio Code launch configuration
    - [x] Think/setup best option for a playground package
    - [x] Configure multiple launch configurations
    - [x] Fight launch specific target keyboard shortcut

## Mon Aug/24

- Emulation/Rust
  - CHIP-8
    - [x] Final cleanups
      - [x] remove NullLogger, and use `Option<Logger>`
      - [x] don't draw screen twice!
    - [ ] Sound interface
      - [x] Think
      - [x] Implement
      - [x] Wire

## Sat Aug/22

- Emulation/Rust
  - CHIP-8
    - [x] Allow screen resize
      - [x] Make window `resizable()`; respond to event in `poll_event()`, and use that in `init()`
      - [x] Manually center canvas
    - [ ] Final cleanups
      - [x] Experiment: Pixel tuple struct

## Fri Aug/21

- Emulation/Rust
  - CHIP-8
    - [x] Implement new algorithm for capping
    - [x] Massive speedup via storing pixel in RGB format
    - [ ] Final cleanups
      - [x] move `set_keys` after `emulate_cycle`

## Thu Aug/20

- Emulation/Rust
  - CHIP-8
    - [x] Add "max speed" mode
    - [x] Refactorings/cleanups
      - [x] replace sdl_context with event_pump
      - [x] split modules instead of using `include!`
      - [x] (failed attempt) EventCode: embed boolean in the KeyCode enums
      - [x] (failed idea): `sdl_to_io_frontend_keycode`: make it enum method (!)
      - [x] Merge wait_keypress and poll_event
      - [x] Pass Quit reference around, rather than returning the value
      - [x] Review `target_texture` purpose
    - [x] fix (game) bugs
        - [x] flightrunner: `PS` doesn't show in `OOPS` anymore
            - backup and remove capping logic
        - [x] wait_keypress doesn't map
        - [x] tombstontipp: closing the window doesn't have any effect
          - wait event steals events from poll events
          - [x] think solution
    - [x] fix refactoring bug
      - [x] tombstone keys are not correct

## Wed Aug/19

- Extra
  - [x] Blog
    - [x] Check out diagram help applications
  - [x] Emulation
    - [x] (Fight) write tool for interactively linting a branch's history

- Emulation/Rust
  - CHIP-8
    - [ ] Refactorings/cleanups
      - [x] Clean errors and (unreported) warnings in history
      - [x] Complete: Review default implementations
    - [x] UI/Frontend improvements
      - [x] implement audio interface
        - [x] implement chip8 beep and test (instruction: `LD ST`)
          - [x] implement speed factor
    - [ ] fix (game bugs)
        - [ ] tombstontipp: closing the window doesn't have any effect
          - wait event steals events from poll events
          - additionally, wait_keypress discards key released events!
        - [ ] flightrunner: `PS` doesn't show in `OOPS` anymore
          - due to screen capping
          - [x] think solution

## Tue Aug/18

- Emulation/Rust
  - CHIP-8
    - [ ] UI/Frontend improvements
      - [x] Allow keys remapping
        - [x] test
      - [x] (Correctly) implement window maximizing and scaling
        - Fix unnecessary clear screen inefficiency
      - [ ] implement audio interface
        - [x] Mad find how to generate beep
        - [x] Fight ceil() function
        - [x] Find pretentious alternative formula
        - [ ] implement chip8 beep
    - [x] Refactor IoInterface: remove draw_pixel
    - [x] Fight Texture/Creator lifetime

## Mon Aug/17

- Emulation/Rust
  - CHIP-8
    - [x] libchip8: Refactoring: Use tuple unpacking for decoding instructions
    - [ ] UI/Frontend improvements
      - [x] Allow user to close the window (and halt the emulator)
        - [x] improve naming
      - [x] Cap display updates to 60 Hz
      - [ ] Allow keys remapping
        - [x] implement
        - [ ] test
    - [ ] Test ROMs/fix bugs
      - [x] flightrunner (crashes after a few runs) -> confirmed rom bug

## Sun Aug/16

- Emulation/Rust
  - CHIP-8
    - [x] Review and hack in video scaling
    - [x] libchip8: Simplify draw sprite routine, and add wrap around
    - [x] Test ROMs/fix bugs (round 1)
      - [x] octojam5title
      - [x] mondrian
      - [x] flightrunner
      - [x] br8kout
      - [x] horseyJump
      - [x] octopaint (unsupported: XO-CHIP)
      - [x] nokiatemplate (runs, although not double checked)
      - [x] ghostescape (runs, but not double checked)
      - [x] 1dcell
      - [x] octojam4title
      - [x] octojam1title
      - [x] sweetcopter (unsupported: Super-CHIP 1.1)
      - [x] octojam3title
    - [ ] UI/Frontend improvements
      - [x] Speed investigation
      - [ ] Allow user to close the window (and halt the emulator)
        - [x] base implementation
    - Quick look at wasm, and source code of related project

## Sat Aug/15

- Extra
  - Scripting
    - Create `brody`
  - Open source
    - Submit PR with shebang addition to CHIP-8 ROMs disassembler project

- Emulation/Rust
  - CHIP-8 base (SDL)
    - [x] implement SDL video/key interface
    - implement remaining chip8 stuff
      - [x] wire `IoFrontend`
      - [x] draw_graphics
      - [x] set_keys
      - [x] instruction 0xFX0A (wait keypress)
      - [x] Think/plan testing/debugging
      - [x] Implement variable screen resolution, and hires mode
      - [x] Fix CALL
      - [x] Implement debug mode
        - fight dynamic traits
        - fight clap
      - [x] Implement debug logs (ASM symbols: http://devernay.free.fr/hacks/chip8/C8TECH10.HTM)
      - [ ] Test via step-in into easy roms, in size order

## Fri Aug/14

- Extra
  - QEMU: build 5.1

- Emulation/Rust
  - CHIP-8 base (SDL)
    - [x] review SDL
      - [x] draw pixels: https://stackoverflow.com/q/20579658
      - [x] (method discarded): draw textures (pixel-based): https://dzone.com/articles/sdl2-pixel-drawing
    - [x] think/implement video/key interface
    - [ ] implement SDL video/key interface

## Thu Aug/13

- Emulation/Rust
  - [ ] review SDL
    - [x] fight event pump
    - [x] fight compiler red herring
    - [x] write improved base program

## Wed Aug/12

- Emulation/Rust
  - CHIP-8
    - implement clock
      - [x] timers
  - [ ] review SDL
    - [x] check guides
      - https://lazyfoo.net/tutorials/SDL
    - [ ] review Rust Programming By Example

- Extra
  - [x] Upgrade Libreoffice; test boot time and check how to improve it

## Tue Aug/11

- Emulation/Rust
  - CHIP-8
    - implement clock
      - [x] research Rust time handling
      - [x] main loop

## Mon Aug/10

- Emulation/Rust
  - [ ] CHIP-8 implementation
    - Base: http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter
      - [x] implement all the opcodes (except 0xFX0A)
    - [Gameloop](https://dewitters.com/dewitters-gameloop) + clock speed
      - [ ] implement clock (~500 Hz; can set no instantiation) + timers

## Sun Aug/09

- Rust: Study
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 2
      - Merge sort (alternative implementations)

## Fri Aug/07

?

## Thu Aug/06

- Emulation/Rust
  - [ ] CHIP-8 implementation

## Wed Aug/05

- Emulation/Rust
  - [ ] Start CHIP-8 implementation
    - Refence: http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter

- Rust
  - Practice
    - Experiment/fight with some ownership concepts (using Clap)

- Extra
  - Find/Review Gitlens extension

## Tue Aug/04

- Rust: Study
  - The Rust Programming Language (completed)
    - [x] Copy #19: Advanced features

- Blog
  - [x] Add Rust book to bookshelf

- Extra
  - [x] Jekyll/Bash scripting

- Emulation
  - Big learning material review and planning
  - [ ] http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter
    - [x] http://en.wikipedia.org/wiki/CHIP-8#Virtual_machine_description

## Mon Aug/03

- Rust: Study
  - The Rust Programming Language
    - [x] Copy #15: Smart pointers
    - [x] Copy #16: Concurrency
    - [x] Copy #17: OO Programming
    - [x] Copy #18: Patterns

## Sun Aug/02

- Rust: Study
  - The Rust Programming Language
    - [x] Copy #12: Project (until skipped)
    - [x] Copy #13: Iterators
    - [ ] Copy #15: Smart pointers
  - Hands-On Data Structures and Algorithms in Rust
    - [x] Section 1 (skipped large part)
  - [x] Fight method chaining on iterators

- Extra
  - Udemy
    - Handle courses/accounts
  - Blog
    - Add AWS course to bookshelf

## Fri Jul/31

- Rust: Study
  - The Rust Programming Language
    - [x] Copy notes #10: Generics, lifetimes

- QEMU-pinning
  - [x] Port patch to 5.1.0-rc2

- Rust: Practice
  - [Ferrous systems teaching exercises](https://ferrous-systems.github.io/teaching-material)
    - [x] [Multithreaded mailbox](https://ferrous-systems.github.io/teaching-material/assignments/multithreaded-mailbox.html)
    - Closing; not interested in the other two exercises
    - Review/copy exercise interesting APIs (if any)

## Thu Jul/30

- Rust: Practice
    - [x] [Connected mailbox](https://ferrous-systems.github.io/teaching-material/assignments/connected-mailbox.html)

## Wed Jul/29

- Rust: Practice
  - [Ferrous systems teaching exercises](https://ferrous-systems.github.io/teaching-material)
    - [ ] [Connected mailbox](https://ferrous-systems.github.io/teaching-material/assignments/connected-mailbox.html)
- Rust: Study
  - The Rust Programming Language
    - [ ] Copy notes #10: Generics, lifetimes

## Tue Jul/28

- Rust
  - Practice
    - [Ferrous systems teaching exercises](https://ferrous-systems.github.io/teaching-material)
      - [x] [Redisish protocol parser](https://ferrous-systems.github.io/teaching-material/assignments/redisish.html)
        - Move to independent repository
        - Review full solution, and convert to library
      - [x] [TCP server](https://ferrous-systems.github.io/teaching-material/assignments/tcp-echo-server.html)
      - [x] [TCP client](https://ferrous-systems.github.io/teaching-material/assignments/tcp-client.html)
  - Study
    - [x] Copy notes #7: Packaging

## Mon Jul/27

- Rust
  - Practice
    - Rustlings
      - One exercise
      - Closing (~60 exercises)
    - Ferrous systems teaching exercises
      - Redisish protocol parser: implementation and tests
        - todo: review full solution, and convert to library

- Work: Fight Ubuntu keyserver

## Sun Jul/26

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Completed remaining sections
    - [x] Completed test exam

## Sat Jul/25

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections: 20

## Fri Jul/24

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections: 12, 13, 15, 16

## Thu Jul/23

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Section 08-09, 17, 19, 21, 10-11
    - [x] Register and setup certification account

## Wed Jul/22

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections 05-07

- Rust
  - Practice
    - Rustlings: couple of exercises

- Extra
  - File VSC search/replace bug

## Tue Jul/21

- Rust
  - Study
    - The Rust Programming Language **top**
      - [x] Complete advanced chapter

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections 01-04

- Extra
  - openscripts
    - `encode_to_m4a`: Add support for (file) symlinks

## Mon Jul/20

- Rust
  - Study
    - The Rust Programming Language **top**
      - [x] Study concurrency chapter
      - Start advanced chapter

- Extra
  - Reset Dropbox shares

## Sun Jul/19

- Extra
  - Scripting

## Sat Jul/18

- Blog: Add Jekyll support for MySQL filtered RSS (2)

- Try Dropbox alternative (https://is.gd/rirV2L)
  - [x] sync: no linux
  - [x] pCloud: problem with docs editing on android
  - [x] zoho docs: no android
  - [x] box.com (10gb): no linux
  - [x] cloudup: invite only
  - [x] spideroak: no free
  - [x] mediafire
  - [x] workzone: no free
  - [x] nextcloud: trash
  - [x] cloudme: poor android app
  - [x] sugarsync: no free
  - [x] teamdrive.com (2gb): terrible android app
  - [x] mega.nz

## Fri Jul/17

- Rust
  - Study
    - The Rust Programming Language
      - Completed reading; skipped some chapters; must copy some
  - Practice
    - Ferrous systems: https://ferrous-systems.github.io/teaching-material
      - [x] [Files, match and Results](https://ferrous-systems.github.io/teaching-material/assignments/result-option-assignment.html)

- Blog: Add MySQL filtered RSS
  - [x] investigate/fix clean jekyll warnings
  - [x] sync gem versions with github pages
  - [x] investigate feasibility (see https://github.com/jekyll/jekyll-feed)
  - [x] fight display icon

- Extra
  - Find temporary Dropbox alternative (-> pCloud) for desktop/tablet

## Thu Jul/16

- Rust
  - Study
    - The Rust Programming Language
      - Follow up
  - Scripting
    - Few updates to `compute_hours`
  - Practice
    - Ferrous systems: https://ferrous-systems.github.io/teaching-material
      - FizzBuzz

- Extra
  - Rust
    - Fight nightly strip binaries

## Wed Jul/15

- ZFS
  - Review fork (limit: Wed/15)

- Rust
  - Study
    - The Rust Programming Language
      - Follow up

- Spreadbase
  - Review and merge PR (limit: Fri/17)

- Extra
  - Find out usable books to use for Rust game development

## Tue Jul/14

- Rust
  - Study
    - The Rust Programming Language
      - Partial #12: Project
      - Start #13: Iterators/closures
  - Practice
    - Rustlings

## Mon Jul/13

- Rust
  - Study
    - The Rust Programming Language
      - Complete #7: Packaging

## Sun Jul/12

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Project #6: Complete -> Course completed

## Sat Jul/11

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Project #6: WIP

- Extra
  - Review Rust study/practice plans

## Fri Jul/10

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Study #6

## Thu Jul/09

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Complete #5
    - Project #5

## Wed Jul/08

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Study #4
    - Project #4
    - Start #5

- ZFS installer
  - Review and discuss proposed feature(s)

## Tue Jul/07

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Project #3

- Extra
  - Replace defective RAM module
  - Review/ask further hardware books

## Mon Jul/06

- Extra
  - New phone
    - Signal test
    - Investigate O/S and rooting

## Sun Jul/05

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Complete Project #2
      - Fight issue with `ALU-nostat.hdl`, and minor other
    - Study #3

- Extra
  - Some Perl

## Sat Jul/04

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Complete project #1
    - Study #2

- Extra
  - Solve dependency issues LLVM repository and i386 packages
  - Try Firefox containers

## Fri Jul/03

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Study #1
    - Start project #1

- Extra
  - Review and reply blog comment
  - Coursera whole setup for studying

## Thu Jul/02

- Rust
  - Practice
    - Rustlings
  - The Rust Programming Language
    - Complete #10 (Generic types, traits, and lifetimes)

- Emulation
  - Computer Architecture: A Quantitative Approach
    - Close #1
    - Start #2
  - Extra: find new book, current is not fit for the purpose

## Wed Jul/01

- Rust
  - The Rust Programming Language
    - Complete #9 (error handling)
    - Start #10

## Tue Jun/30

- Rust
  - The Rust Programming Language
    - Follow up #9 (error handling)

- ZFS
  - Add Linux Mint 20 support

- Extra
  - Port QEMU patch to 4.2.1

## Sun Jun/28

- Emulation
  - Computer Architecture: A Quantitative Approach

## Sat Jun/27

- Rust
  - fork, clean, and submit Rust programming by example source code

## Fri Jun/26

- Rust
  - Rust programming by example
    - Follow up #3

## Mon Jun/22

- Rust
  - Scripting
    - Completed hours compute script

## Sun Jun/21

- Rust
  - Rust programming by example
    - Complete #1, #2
  - Scripting
    - Write working hours compute script
      - Review argument parsing in Rust
      - Fight borrow checker

- Extra
  - Test vivaldi (https://help.vivaldi.com/article/manual-setup-vivaldi-linux-repositories)

## Mon Jun/15

- Extra
  - Blog
    - Completed article on WFs/CTEs; took also few days during last week

## Sun Jun/07

- Rust
  - Study #8
  - Start #9

- Rust/Golang
  - Review/plan books and practice

- Extra
  - Think future article about apt/snaps/ppas
  - Read index dives article

## Sat Jun/06

- Golang
  - Powerful Command-Line...
    - Complete copy and review #4
    - Copy and review #5, #6

- Extra
  - Scripting: Fix tomatoes script

## Fri Jun/05

- Extra
  - Blog
    - Whole day research on user-defined variables problem

## Tue Jun/02

- Golang
  - Powerful Command-Line...: Complete the remaining chapters -> ?

## Fri May/29

- Golang
  - Read article on (XKCD) CLI application
  - Powerful Command-Line...: (Finish) study #4

## Thu May/28

- Golang
  - Powerful Command-Line Applications in Go: Copy and review #3
  - Exercism: One exercise
    - Fight parallelism and multiple arrays

- Extra
  - ZFS
    - Diagnose reported problem, and create minimal test case
    - Improve disks wiping, by using the zpool functionality
    - Ask ML resizing safety
    - Review zvols, and reply on Launchpad
  - delete_in_batches
    - review, and send analysis to project

## Wed May/27

- Golang
  - Powerful Command-Line Applications in Go: Copy and review #2
  - Research FlagSet

- Extra
  - Mirror and create PPA for [ppastats](https://fosspost.org/tutorials/how-to-get-the-download-stats-of-any-ubuntu-ppa)
  - ZFS: Close issue, with PR `Warn about "-O devices=off" already set in the tweaks dialog`
  - ZFS: Investigate pool corruption issue and propose solution
  - Desktop: Correct MIME types association

## Tue May/26

- Extra
  - PPA: Push scripts to new repository
  - Blog: Complete and release PPA article
  - PPA: Minor tweaks to published scripts

## Mon May/25

- Extra
  - PPA: Complete scripting infrastructure

## Sun May/24

- Golang
  - Review gccgo (+alias)
  - Study Study: Powerful Command-Line Applications in Go, until Chapter 3

## Sat May/23

- Golang
  - Re-read chapter 16
  - Study chapter 19

- Extra
  - Browser scripting
    - Rewritten logic; hopefully final version.
  - Books
    - Fight Dropbox not syncing tablet book(s)
  - Fight tablet SMB connection
  - Fight display sleep on laptop
  - Write article on display sleep

## Fri May/22

- Extra
  - PPA
    - Expand and prepare tooling for internal production usage
    - Add cowbuild option

## Thu May/21

- Rust
  - Review/copy notes chapter 6/beginning 8
  - Rustlings

- Extra
  - PPA
    - Update script
    - Review and reorganize documentation
  - Scripting
    - Write daily notes template script

## Wed May/20

- Rust
  - Review/copy notes chapter 5

- Extra
  - Exercism discussion

## Tue May/19

- Extra
  - (Mad) PPA Ruby packaging

## Mon May/18

- Extra
  - ZFS
    - Reply email
  - (Mad) PPA Ruby packaging

## Sat May/16

- Rust
  - Review notes chapter 4

- Extra
  - System
    - Correct xbacklight support on laptop
    - Finally solve issue with backlight

## Fri May/15

- Rust
  - Several Rustlings exercises

- Extra
  - ZFS installer
    - Review and fix issue

## Thu May/14

- Rust
  - Several exercism exercises
  - Review resources for following up practice

- Extra
  - Fix/extract repositories sync script

## Wed May/13

- Rust
  - One Exercism exercise (~1h)

- Extra
  - Update exercism management script (~30')
    - Workaround submission "playgrounded" scripts
    - Automatically patch Rust test file(s)

## Tue May/12

- Rust
  - Resume studies

- Extra
  - Update exercism management script (~1.5h)
    - Fight bash inherit_errexit

## Mon May/11

- Rust
  - Resume studies

- Extra
  - Scripts
    - `notify_travis_builds`: Make compatible with Ruby 2.7
  - Geet
    - First Ruby 2.7 upgrade work
    - Add Ruby 2.7 and HEAD to build
  - Bash
    - Nightmare bash RC files

## Sun May/10

- Extra
  - Review/copy Git notes (`log`)

## Sun May/09

- Extra
  - System
    - Solve permanently screen turning off spontaneously after issuing `xset dpms force off` 

## Sat May/08

- Extra
  - System
    - Update desktop to Ubuntu 20.04
    - Handle VFIO on 20.04
    - Update vga-passthrought project to 20.04

## Fri May/07

- Extra
  - System
    - Move to new password manager
    - Fight NxServer issue; report

## Wed May/06

- Extra
  - ZFS
    - Complete shortcut script for fast O/S installation
  - QEMU-pinning
    - Test v5.0.0 patch
  - Firefox scripting
    - Add support for slack:// protocol

## Tue May/05

- Extra
  - ZFS
    - Prepare animated demo
    - Make raid type user selectable
    - Reorganize pools creation/custom script execution
    - Add script for fast O/S installation via partition snapshot

## Mon May/04

- Extra
  - Dconf/Gsettings review
  - ZFS installer
    - Manage RAIDZ contribution
  - Find strategy for capturing video and encoding it, with skips

## Sat May/02

- Extra
  - Minor shell scripting

## Thu Apr/30

- Extra
  - Finalize Firefox launcher script
  - 20.04/laptop
    - First look of S3 client alternatives
    - Fight video crashing
    - Fight set default/preferred application

## Wed Apr/29

- Extra
  - Upgrade laptop to Ubuntu 20.04
  - Write super-annoying Firefox launcher script

## Tue Apr/28

- Extra
  - ZFS installer
    - Review PR, issue
    - Cool mkfifo solution for not logging confidential information
    - Debug desktop env not logged
    - Add new logging
    - Release new installer version, and reply issue
  - QEMU pinning
    - Updated patch; required updates (needs testing)

## Mon Apr/27

- Extra
  - Ruby (notes/review): ampersand prefix operator
  - Date filtering extension to internal Trello management tool
  - System: move passwords to new password manager
  - System: big update email client

## Sun Apr/26

- Extra
  - ZFS installer
    - Improve UX (and code) for the passphrase configuration
    - Considerable fight against Ubiquity 20.04
    - Insane amount of overall work, including final support for Ubuntu Desktop/Server 20.04
- Copy some linux notes; experiment with resizing/partitions

## Sat Apr/25

- Extra
  - ZFS installer: lots of work, mostly related to Ubuntu 20.04

## Fri Apr/24

- Extra
  - Ubuntu 20.04 preparation, part 2
  - ZFS installer
    - Close 1 issue (confidential information in the log)
    - Add extra logging
      - Fight variables in sudo env
    - Fix shellcheck warnings
    - PRs management/review
  - Copy some Linux system notes, reviewing Apt stuff

## Thu Apr/23

- Mentoring
  - 1 iteration

## Tue Apr/21

- Rust
  - Study chapters 5, 6

- Mentoring
  - 3 iterations
  - 1 solution

## Mon Apr/20

- Golang
  - Experiment/copy notes remaining chapter 16 (Concurrency)

- Mentoring
  - 9 solutions

- Extra (≈1h)
  - Several reviews on the Goby project
  - Review clever regex for finding sequences

## Sun Apr/19

- Golang
  - Re-read chapter 16
  - Study chapter 17
  - Copy notes chapter 8, 17; half chapter 16

- Extra
  - Tweaks to broser script + test multiple windows matching behavior with xdotool

## Fri Apr/17

- Golang
  - Review and copy notes chapters 7, 9
  - Study chapter 8, 9

- Extra
  - Ruby: review process execution strategies

## Thu Apr/16

- Golang study
  - Study chapters 5, 6, 7
    - Copy notes chapters 5, 6
  - 2 exercises exercism

- Mentoring
  - 2 iterations
  - two exercism discussions

- Extra
  - Bash: exact filenames cycling, with complex text, and handling inside function
  - Review Golang exercise website options
    - Register gophercises

## Wed Apr/15

- Golang study
  - Copied/reviewed chapter 4

- Mentoring
  - Discussion about ratings
  - 6 iterations

- Extra
  - checked out the Zeus source code for signals/restart support
  - tested spring in explicit mode

## Tue Apr/14

- Extra
  - MySQL `REGEXP_SUBSTR()` and review regex capabilities
  - Bash: for/while + IFS (fight while)
  - Check out Golang GUI libraries/book

- Mentoring
  - Around 15 iterations (≈1h)

## Mon Apr/13

- Golang study
  - Complete chapters 2,3
    - Merged previous other book notes
  - Exercises
    - Bubble sort

- Extra
  - Preparation installation Ubuntu 20.04 (#1)

## Sun Apr/12

- Rust study
  - Complete chapter 4
    - Copy notes, including previous chapters
  - Fight exercism exercise

- Mentoring
  - Around a dozen exercism iterations
  - Week total: 35

- Extra
  - Big refactoring and extension of the Exercism script

## Sat Apr/11

- Rust study
  - New APIs, from exercises
  - 1 exercism exercise
  - Follow up chapter 3, start chapter 4

- Mentoring
  - 3/4 Bash iterations

- Extra
  - Expand Exercism script: open and submit

## Fri Apr/10

- Rust study
  - Little study
  - Exercise: fight `&str` vs. `to_string()`

## Thu Apr/09

- Rust study
  - Copy book notes
  - Follow up chapter 3
  - 1 exercism exercise

- Extra
  - ZFS installer: increase boot pool size to 768M

- Mentoring
  - ≈5 Ruby iterations
  - 1 Bash iteration

## Wed Apr/08

- Rust study
  - put book concepts into small program
  - 1 exercism exercise

- General notes
  - Flush remaining pending notes (Bash/Linux tools)

- Extra
  - Unf#ck Slack email links
  - Blog post about Unf#cking Slack email links (part 1)
    - Minor review of Bash regexes

## Tue Apr/07

- Mentoring
  - 7/8 Ruby iterations

- General notes
  - copy/review some Bash subjects, mainly process substitution
  - copy minor Linux tools/Ruby subjects

## Mon Apr/06

- General notes
  - copy/reorganize several Linux subjects

- Rust study
  - followed up chapter 2

- Mentoring
  - Review four Ruby solutions

- Extra
  - Make browser scripts put Firefox window in front when launching from Thunderbird
  - Add custom browser desktop application, and associate it

## Sun Apr/05

- Exercises: Golang
  - Many exercism exercises
    - Spent considerable time on a regex-based exercise 
    - Spent time on byte-level strings (/slices) handling

- Mentoring
  - Review several (Ruby) solutions

- Extra
  - Discussions on Exercism Slack channels

## Sat Apr/04

- Mentoring
  - Review several solutions

- Extra
  - Emergency potential malware on Chromium
    - Move from Chromium to Firefox (lots of work)
  - VSC snippets

## Fri Apr/03

- Study
  - Golang: several codewars kata
  - Golang: three exercism

- Mentoring
  - Several solutions (with an interesting concept found in one)

## Thu Apr/02

- Study
  - Golang: 3 codewars kata

- https://mast.queensu.ca/~peter/inprocess/sumofcubes.pdf
  - `sqrt(sum) = (n² + n) / 2`
- https://www.tiger-algebra.com/drill/n~2_n-200=0
  - `An2+Bn+C = 0 -> (-B ± sqrt(B²-4AC)) / 2A`

## Wed Apr/01

- Extra
  - think plan Exercism; review Codewars

- Waste
  - 15' wiki lino banfi/franco e ciccio

## Tue Mar/31

- Mentoring
  - Five solutions/iterations

- Study
  - Copy notes: Golang
  - Copy notes: Rust

- Extra
  - Bash: one exercise (left track)
  - Copy notes: generic
    - tested and reorganized Linux parallel tools

## Mon Mar/30

- Mentoring
  - Some solutions
  - Long reply on glob expansion
  - Difficult regex on one

- Study
  - Golang: few exercises
    - One took a very long time

- Extra
  - Bash: one exercise
    - Review substring functionalities
  - Bash: research glob expansion
  - Try vscode golang debug

- Waste
  - Many minor distractions over the day
  - HN 30' reply

## Sun Mar/29

- Study
  - Golang: a couple of exercisms
- Mentoring
  - Several solutions, including one complex and long-standing
- Extra
  - Written script for managing exercism
  - Bash: a couple of exercisms

## Sat Mar/28

- Study
  - Golang: 1 chapter
  - Rust: one exercism
- Mentoring
  - A complex and long-standing solution

## Fri Mar/27

- Scripting
  - Trello/notify_travis_builds
- Exercism
  - A few solutions
  - Started a snippets file with common concepts/replies
- Notes
  - Copied/moved a few Bash ones
- Studies
  - First chapter golang
  - One exercism golang

- Extra/waste
  - Install golang 1.14
  - Review Codewars with Stan
  - Additional exercism session
  - Update libreoffice
