# Rust Performance

- [Rust Performance](#rust-performance)
  - [User-facing](#user-facing)
    - [std::mem APIs](#stdmem-apis)
    - [Unsafe](#unsafe)
    - [Uninitialized memory (`std::mem::MaybeUninit`)](#uninitialized-memory-stdmemmaybeuninit)
      - [Bidimensional array struct, uninitialized and macro'ed](#bidimensional-array-struct-uninitialized-and-macroed)
    - [std::num::NonZero*](#stdnumnonzero)
    - [(LLVM) Intrinsics/ASM APIs](#llvm-intrinsicsasm-apis)
    - [Embedded/OS development](#embeddedos-development)
  - [Internal](#internal)
    - [Memory layout](#memory-layout)
    - [References](#references)
    - [Null pointer optimization](#null-pointer-optimization)

## User-facing

### std::mem APIs

```rs
size_of::<Type>()         // memory occupation of a type
size_of_val(&v)           // memory occupation of a variable
unsafe { zeroed::<T>() }  // return zeroed memory for type; it's unsafe because zero(s) may not be a valid value
```

### Unsafe

Raw pointers:

```rust
fn print_raw_pointers(ref_mut: *mut i32, ref_const: *const i32) {
  unsafe {
    println!("r1:{}, r2:{}", *ref_const, *ref_mut);
  }
}

let mut num = 5;

// Creating pointers is not unsafe - only dereferencing them.
// They don't own the data, and the multiple raw pointers to the same data are valid.
//
let ref_mut = &mut num as *mut i32;
let ref_const = &num as *const i32;
let ref_const_2 = &num as *const i32;

print_raw_pointers(ref_mut, ref_const);

// Invalid, due to BC; as workaround, pass the raw pointer as separate variable.
//
mymethod(&mut num, &num as *const i32);
```

Functions:

```rust
// Unsafe function; unsafe calls themselves don't require the enclosing function to be marked as such.
//
unsafe fn dangerous() {}

// Unsafe funcion call
//
unsafe {
  dangerous();
}
```

Example: slicing an array. It can't be done safely, because we can't mutably borrow twice.

```rust
let slice = &mut [0, 1, 2, 3][..];
let mid = 2;

let len = slice.len();
let ptr = slice.as_mut_ptr();

let (slice1, slice2) = unsafe {
  (
    std::slice::from_raw_parts_mut(ptr, mid),
    std::slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid),
  )
};
```

Referencing raw memory (undefined behavior):

```rust
let address = 0x012345_usize;
let r = address as *mut i32;

let slice: &[i32] = unsafe {
  std::slice::from_raw_parts_mut(r, 10000)
};
```

### Uninitialized memory (`std::mem::MaybeUninit`)

Safe(r) version of initialization with uninitialized values of Array and i32; the simpler but unsafe(r) versions are below.

```rs
let mut result: [MaybeUninit<crate::Color>; pixels_count] = unsafe { MaybeUninit::uninit().assume_init() };
result[0] = MaybeUninit::new(crate::Color { r: 0.0, g: 0.0, b: 0.0 });
let result = unsafe { mem::transmute::<_, [crate::Color; pixels_count]>(result) };

let mut x = MaybeUninit::<&i32>::uninit();
unsafe { x.as_mut_ptr().write(&0); }
let x = unsafe { x.assume_init() };
```

For a more sophisticated application to bidimensional arrays, see the [subsection below](#bidimensional-array-struct-uninitialized-and-macroed).

Other uninitialized forms:

```rs
let mut result: [[f64; $order]; $order] = usafe { mem::zeroed() };
let mut result: [[f64; $order]; $order] = usafe { MaybeUninit::zeroed().assume_init( }) // same as previous

let mut result: [[f64; $order]; $order] = usafe { mem::uninitialized() };
let mut result: [[f64; $order]; $order] = usafe { MaybeUninit::uninit().assume_init( }) // same as previous
```

There are also more sophisticated uses, e.g. partially initialized arrays.

#### Bidimensional array struct, uninitialized and macro'ed

For the lulz; in such cases, avoiding the MaybeUninit intermediate value is simpler.

```rust
macro_rules! matrix {
  ($name:ident, $order: literal) => {
    #[derive(Debug)]
    pub struct $name {
      pub values: [[f64; $order]; $order],
    }

    impl $name {
      pub fn new(source_values: &[f64]) -> Self {
        let mut dest_values: [MaybeUninit<[f64; $order]>; $order] = unsafe { MaybeUninit::uninit().assume_init() };

        for (dest_row, source_row) in dest_values.iter_mut().zip(source_values.chunks($order)) {
          *dest_row = MaybeUninit::new(source_row.try_into().unwrap());
        }

        let dest_values = unsafe { mem::transmute::<_, [[f64; $order]; $order]>(dest_values) };

        $name { values: dest_values }
      }
    }
  };
}
```

Simpler/unsafer logic:

```rust
let mut values: [[f64; $order]; $order] = unsafe { MaybeUninit::uninit().assume_init() };

for (row, source_row) in values.iter_mut().zip(source_values.chunks($order)) {
  row.copy_from_slice(source_row);
}
```

### std::num::NonZero*

If an integer variable is guaranteed never to be 0, one can use `NonZero*` data types, which allow some optimizations.

### (LLVM) Intrinsics/ASM APIs

Intrinsic/ASM APIs are unstable.

```rs
#![feature(core_intrinsics)]  // Enable the (unstable) intrinsics APIs
#![feature(link_llvm_intrinsics)]
#![feature(asm)]              // Enable the (unstable) ASM APIs

use core::intrinsics;

unsafe { intrinsics::abort(); }

extern "C" {
    // Provides specific instructions to the linker about where it should look to find the function
    // definitions
    #[link_name = "llvm.eh.sjlj.setjmp"]
    pub fn setjmp(_: *mut i8) -> i32;
}

loop {
    unsafe { asm!("hlt"); }
}
```

### Embedded/OS development

Basic structure for an O/S program:

```rs
#![no_std]                    // Don't include the stdlib.
#![no_main]                   // Program doesn't have main()
#![feature(core_intrinsics)]
#![feature(asm)]

use core::intrinsics;
use core::panic::PanicInfo;

#[panic_handler]
#[no_mangle]
pub fn panic(_info: &PanicInfo) -> ! {
    unsafe { intrinsics::abort(); }
}

// [lang]uage items are elements of Rust implemented as libraries outside of the compiler itself; in
// this context, we don't include the stdlib.
// [E]xception [H]andler. "Personality" routines are executed at each step of the call stack unwinding.
//
#[lang = "eh_personality"]
#[no_mangle]
pub extern "C" fn eh_personality() {}

#[no_mangle]
pub extern "C" fn _start() -> ! {
    let mut framebuffer = 0xb8000 as *mut u8;

    unsafe {
        // Using write_volatile() prevents the compiler from performing certain optimizations, e.g.
        // presetting the memory before execution rather than writing during execution.
        framebuffer.offset(1).write_volatile(0x30);
        // Alternate write form.
        *(framebuffer + 1) = 0x30;
    }

    // Never return, as there's nowhere to return to.
    loop {
        unsafe { asm!("hlt"); }
    }
}
```

## Internal

### Memory layout

Structs without fields are "Zero-sized types" (ZST), and don't take any space in the resulting program.

### References

Reference:

- are pointers; for dynamically sized types, they also include an integer (with extra information)
- are aligned to `usize`

### Null pointer optimization

With an enum like this:

```rs
enum MyEnum {
  EmptyVariant
  Variant<NeverZeroType>
}
```

internally, `EmptyVariant` will be stored as `Variant` with zero bits, making the enum data structure take no space.
