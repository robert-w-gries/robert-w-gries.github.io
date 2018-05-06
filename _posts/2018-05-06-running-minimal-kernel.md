---
title: "Migrating to a Rust Bootloader: Running a Minimal Kernel"
---

This post is the first of a two-part series where we will migrate your Rust kernel from using the GRUB bootloader to a [Rust bootloader](https://github.com/rust-osdev/bootloader).

In this post, we will implement the [Minimal Rust Kernel](https://os.phil-opp.com/minimal-rust-kernel/) post by Phil Oppermann for existing Rust kernels.

I will assume that the reader has implemented the [first edition](https://os.phil-opp.com/) and read the [second edition](https://os.phil-opp.com/second-edition/) of Phil's blog.

# Forked `blog_os`

To make it easy for readers to follow along with this post, I forked the `blog_os` kernel and created a `first_edition` branch that compiles for the latest `rustc` nightly.

```bash
git clone -b first_edition https://github.com/robert-w-gries/blog_os.git
```

However, building this branch will almost certainly fail after API breakages in future Rust compiler updates.  So let's override the nightly version so that this branch will always build.

```bash
rustup override set nightly-2018-04-30
```

# Building a Rust Binary

The first edition of `blog_os` builds the kernel as a static library because we needed to link assembled object files with our Rust code.  These assembled files specified support for the Multiboot standard, set up long mode, and jumped to our Rust entry point.

The `bootimage` tool removes the need to manually link our kernel.  It instead compiles our kernel to an `ELF` file and appends the bytes of the `ELF` file to the bootable `rust-osdev/bootloader` binary.

There are a few changes we need to make to our project to build a binary instead of a static library.

## Target Specification

We need to update some fields in our target specification file.  Detailed information for all fields can be found [here](https://doc.rust-lang.org/1.0.0/rustc_back/target/struct.TargetOptions.html).

* `llvm-target: "x86_64-unknown-none"`
  * Our code will run on bare metal, so we change this field to `x86_64-unknown-none`.
* `linker-flavor: "ld.lld"`
  * Switching to the [LLD](https://lld.llvm.org/) is not **needed** but it is a nice change because the LLD linker is cross-platform.
* `executables: true`
  * This field specifies that executables are available on the target. Needed to build a binary instead of a static library.

```diff
diff --git a/x86_64-blog_os.json b/x86_64-blog_os.json
index 909d3fe..9eb3881 100644
--- a/x86_64-blog_os.json
+++ b/x86_64-blog_os.json
@@ -1,7 +1,7 @@
 {
-  "llvm-target": "x86_64-unknown-linux-gnu",
+  "llvm-target": "x86_64-unknown-none",
   "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
-  "linker-flavor": "gcc",
+  "linker-flavor": "ld.lld",
   "target-endian": "little",
   "target-pointer-width": "64",
   "target-c-int-width": "32",
@@ -9,5 +9,6 @@
   "os": "none",
   "features": "-mmx,-sse,+soft-float",
   "disable-redzone": true,
-  "panic": "abort"
+  "panic": "abort",
+  "executables": true
 }
```

## `Cargo.toml`

We also need to update the `Cargo.toml` file so that it stops packaging the kernel as static library.

```diff
diff --git a/Cargo.toml b/Cargo.toml
index 701395b..50e00ec 100644
--- a/Cargo.toml
+++ b/Cargo.toml
@@ -17,6 +17,3 @@ linked_list_allocator = "0.6.0"
 [dependencies.lazy_static]
 features = ["spin_no_std"]
 version = "0.2.1"
-
-[lib]
-crate-type = ["staticlib"]
```

## Rust Binary Entry Point

Now our tools know we are building a binary instead of a static library.  One more change needed for the build tools is to rename our `src/lib.rs` file to `src/main.rs`.

```bash
mv src/lib.rs src/main.rs
```

Let's see what happens when we try to run `bootimage`.

```bash
# To install bootimage, run `cargo install bootimage`
bootimage run --target x86_64-blog_os
Building kernel
   Compiling bit_field v0.7.0
   Compiling spin v0.4.8
   Compiling bitflags v0.4.0
   Compiling bitflags v0.9.1
   Compiling once v0.3.3
   Compiling volatile v0.1.0
   Compiling rlibc v1.0.0
   Compiling x86_64 v0.1.2
   Compiling lazy_static v0.2.11
   Compiling linked_list_allocator v0.6.1
   Compiling multiboot2 v0.1.0
   Compiling blog_os v0.1.0 (file:///home/rob/projects/blog_os)
warning: unused `#[macro_use]` import
  --> src/main.rs:22:1
   |
22 | #[macro_use]
   | ^^^^^^^^^^^^
   |
   = note: #[warn(unused_imports)] on by default

error[E0601]: `main` function not found in crate `blog_os`
  |
  = note: consider adding a `main` function to `src/main.rs`

error: aborting due to previous error

```

We can see that the rust compiler expects a `main()` function in `src/main.rs`.

But as Phil explains in the [Freestanding Rust Binary](https://os.phil-opp.com/freestanding-rust-binary/#the-1) post, we do not have access to the Rust runtime, which calls `main()` after performing some minor setup.  Instead, we need to overwrite the entry point to our binary.  It appears that LLVM uses Linux conventions by default, so we will need to name the entry point `_start` specifically.

### `#![no_main]`

First, we have to indicate to the compiler that we will not use a `main()` function as our Rust 's entry point.

```diff
diff --git a/src/main.rs b/src/main.rs
index 9a083f1..ac6c283 100644
--- a/src/main.rs
+++ b/src/main.rs
@@ -18,6 +18,7 @@
 #![feature(global_allocator)]
 #![feature(ptr_internals)]
 #![no_std]
+#![no_main]

 #[macro_use]
 extern crate alloc;
```

### `_start`

Second, we have to create a `_start()` function in `src/main.rs`.  We can repurpose `rust_main()` as our entry point.  Note that I removed the pointer to `multiboot_information` because the `rust-osdev/bootloader` does not use Multiboot.

```diff
diff --git a/src/main.rs b/src/main.rs
index ac6c283..6b39ecb 100644
--- a/src/main.rs
+++ b/src/main.rs
@@ -41,7 +41,7 @@ mod memory;
 mod interrupts;

 #[no_mangle]
-pub extern "C" fn rust_main(multiboot_information_address: usize) {
+pub extern "C" fn _start() -> ! {
     // ATTENTION: we have a very small stack and no guard page
     vga_buffer::clear_screen();
     println!("Hello World{}", "!");
```

# Running a Minimal Kernel

We build a Rust binary now and overwrite the entry point to point to our Rust code.  Let's see if we can run our kernel with `bootimage` now.

```bash
bootimage run --target x86_64-blog_os
Building kernel
   Compiling spin v0.4.8
   Compiling bitflags v0.4.0
   Compiling bit_field v0.7.0
   Compiling volatile v0.1.0
   Compiling once v0.3.3
   Compiling rlibc v1.0.0
   Compiling bitflags v0.9.1
   Compiling linked_list_allocator v0.6.1
   Compiling lazy_static v0.2.11
   Compiling multiboot2 v0.1.0
   Compiling x86_64 v0.1.2
   Compiling blog_os v0.1.0 (file:///home/rob/projects/blog_os)
error[E0425]: cannot find value `multiboot_information_address` in this scope
  --> src/main.rs:49:47
   |
49 |     let boot_info = unsafe { multiboot2::load(multiboot_information_address) };
   |                                               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ not found in this scope

warning: unused `#[macro_use]` import
  --> src/main.rs:23:1
   |
23 | #[macro_use]
   | ^^^^^^^^^^^^
   |
   = note: #[warn(unused_imports)] on by default

error: aborting due to previous error

For more information about this error, try `rustc --explain E0425`.
error: Could not compile `blog_os`.

To learn more, run the command again with --verbose.
```

Of course the kernel doesn't build, we just removed a crucial piece of information: the `multiboot_information_address`!  In this post, we merely want to get the kernel running with the new bootloader, so let's comment out all non-printing code:

```diff
diff --git a/src/main.rs b/src/main.rs
index 6b39ecb..e4a6231 100644
--- a/src/main.rs
+++ b/src/main.rs
@@ -46,26 +46,26 @@ pub extern "C" fn _start() -> ! {
     vga_buffer::clear_screen();
     println!("Hello World{}", "!");

-    let boot_info = unsafe { multiboot2::load(multiboot_information_address) };
-    enable_nxe_bit();
-    enable_write_protect_bit();
+    //let boot_info = unsafe { multiboot2::load(multiboot_information_address) };
+    //enable_nxe_bit();
+    //enable_write_protect_bit();

     // set up guard page and map the heap pages
-    let mut memory_controller = memory::init(boot_info);
+    //let mut memory_controller = memory::init(boot_info);

-    unsafe {
-        HEAP_ALLOCATOR.lock().init(HEAP_START, HEAP_SIZE);
-    }
+    //unsafe {
+    //    HEAP_ALLOCATOR.lock().init(HEAP_START, HEAP_SIZE);
+    //}

     // initialize our IDT
-    interrupts::init(&mut memory_controller);
+    //interrupts::init(&mut memory_controller);

-    fn stack_overflow() {
-        stack_overflow(); // for each recursion, the return address is pushed
-    }
+    //fn stack_overflow() {
+    //    stack_overflow(); // for each recursion, the return address is pushed
+    //}

     // trigger a stack overflow
-    stack_overflow();
+    //stack_overflow();


     println!("It did not crash!");
```

If we run the kernel now, we will see that we can successfully build and run the kernel with no crashes!

![image](/assets/images/running_minimal_kernel.png)

# What's Next?

We can successfully build and run our kernel with the `rust-osdev/bootloader`!  But we had to remove nearly all functionality from the kernel to do so.

In the next post, we will re-write the memory module to use information passed from the new bootloader and add all functionality back to our kernel!
