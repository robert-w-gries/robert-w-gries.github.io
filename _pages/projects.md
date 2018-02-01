---
permalink: /projects/
layout: single
author_profile: true
---

Here are some projects and open source contributions that display my versalitiy.

# Industry Work

## Puppet/Chef Modules | [Cisco](https://www.cisco.com/)

#### [Node Utils](https://github.com/cisco/cisco-network-node-utils)

Ruby abstraction layer to facilitate management of Cisco datacenter switches. Used in Cisco's Puppet and Chef modules.

#### [ciscopuppet](https://github.com/cisco/cisco-network-puppet-module)

[Puppet](https://puppet.com/) module for Cisco switches.

#### [cisco-cookbook](https://github.com/cisco/cisco-network-chef-cookbook)

[Chef](https://www.chef.io/) cookbook for Cisco switches.

## ASR 9000 Multicast Microcode | [Cisco](https://www.cisco.com/)

Developed the microcode of a new architecture for Cisco's flagship Service Provider edge router.

## Thermotron Serial I/O | [Continental Automotive](https://www.continental-automotive.com/)

As an Embedded Software Engineer Intern, I extended an automated testing framework to interface with Thermotron chambers through serial commands.  This was my first project in my career as a Software Engineer.

# Open Source Contributions

## rXinu

[Read more about rXinu here](/rxinu/)

## [blog_os](https://github.com/phil-opp/blog_os) - Rust Kernel

>This blog series creates a small operating system in the Rust programming language. Each post is a small tutorial and includes all needed code, so you can follow along if you like.

I am both a contributor and a user of this repository.  I followed the blog series to create a kernel with memory allocation and interrupt handling. This minimal kernel formed the foundation of rXinu.

In order to give back to a project so integral to the development of `rXinu`, I have submitted numerous patches and resolved open issues.

## [multiboot2-elf64](https://github.com/rust-osdev/multiboot2-elf64)

A Multiboot2 library written in Rust.  This crate is generally used in Rust kernels to read basic information passed by a multiboot2 compliant bootloader.

As a collaborator, I assist in maintaining this project and ensuring the latest nightly Rust compiler is supported.

## [linked-list-allocator](https://github.com/phil-opp/linked-list-allocator)

>Simple allocator usable for no_std systems. It builds a linked list from the freed blocks and thus needs no additional data structures.

This memory allocator is used in rXinu as a simple allocator that can be setup with minimal overhead.  As a contributor, I have [submitted an enhancement](https://github.com/phil-opp/linked-list-allocator/pull/7) to improve the handling of an Out Of Memory kernel panic.

# School Projects

## Educational Network Emulator

## Xinu

## MiniJava Compiler
