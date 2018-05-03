---
title: Migrating From GRUB to Pure Rust Bootloader
---

This post is a guide to switching your Rust kernel from the GRUB bootloader to Phil Oppermann's [Rust bootloader](https://github.com/rust-osdev/bootloader).  I will assume that the reader has implemented most, if not all, of the first edition of Phil's [blog](https://os.phil-opp.com/).

If you wish to learn more about the pure Rust bootloader, see Phil's series of blog posts [here](https://os.phil-opp.com/second-edition/).

# Forked `blog_os`

To make it easy for readers to follow along with this post, I forked the `blog_os` kernel and created a `first_edition` branch that compiles for the latest `rustc` nightly.

```bash
git clone -b first_edition https://github.com/robert-w-gries/blog_os.git
```

However, building this branch will almost certainly fail after API breakages in future Rust compiler updates.  So let's override the nightly version so that this branch will always build.

```bash
rustup override set nightly-2018-04-30
```

# Why switch to the Rust bootloader?



# A Minimal Rust Kernel

Our first step is to implement Phil's post on setting up a [Minimal Rust Kernel](https://os.phil-opp.com/minimal-rust-kernel/).  The changes needed are fairly straightforward.

## Configuration Files

First, let's modify our configuration files so we are telling the compiler to build an executable binary.  We need to change our target specification file, `x86_64-blog_os.json`, and `Cargo.toml`.

### Target Specification

Detailed information on all fields can be found [here](https://doc.rust-lang.org/1.0.0/rustc_back/target/struct.TargetOptions.html).

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

Since we want to build a binary instead of a static library, we need to remove the `crate-type = ["staticlib"]` line.

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

## `src/main.rs`

We now build a binary instead of a static library.  This means that we have to rename our `src/lib.rs` file to `src/main.rs`.

```bash
mv src/lib.rs src/main.rs
```
