---
permalink: /projects/
layout: single
author_profile: true
toc: true
---

Here are some projects and open source contributions that show my versalitiy.

## Durham Food Resources (Android App)

| GitHub Repos | Description |
|--------------|-------------|
| [pantries-android](https://github.com/end-hunger-durham/pantries-android) | Android app; available on Google Play |
| [pantries-ios](https://github.com/end-hunger-durham/pantries-ios) | iOS app; available on the Apple App Store |

My participation in [Code for Durham](https://codefordurham.com) led to my involvement with [End Hunger Durham](www.endhungerdurham.org).

This collaboration has resulted in an Android and iOS app that displays food pantries in the Durham, NC area. The information provided includes location, phone numbers, and times of availability. The availability information is especially crucial as pantries do not have consistent hours.

Anyone can use the app, but we have found two core target user groups exist: people in need of emergency food and service workers who connect clients in need to available food providers.

With the recent Covid-19 outbreak, the pantry information has been rapidly changing with popup emergency stations and temporary closures. During this time, the app has been frequently updated to reflect the latest changes.

## Cisco Open Source Work

| GitHub Repos | Description |
|-------------------|-------------|
| [cisco-network-node-utils](https://github.com/cisco/cisco-network-node-utils) | Abstraction library to support both Puppet and Chef modules. |
| [ciscopuppet](https://github.com/cisco/cisco-network-puppet-module) | Puppet module for Cisco datacenter switches. |
| [cisco-cookbook](https://github.com/cisco/cisco-network-chef-cookbook) | Chef module for Cisco datacenter switches. |

As a member of the Cloud & Virtualization team at [Cisco](https://www.cisco.com/), I wrote orchestration software that automated the configuration of datacenter switches and serivce routers.  Generally, configuration of networking elements is a tedious and non-trivial problem.  With orchestration tools, this task is simpler to manage and easier to push at scale.

This functionality was achieved by writng [Puppet](https://puppet.com/) and [Chef](https://www.chef.io/) modules that were installed on either the native Linux enviornment or a CentOS container running on the Cisco product.

My work on this project included the follwoing:

* Add support for automation of Cisco features to the abstraction layer, the Chef cookbook, and the Puppet module.
* Write a Continuous Integration system that would build the project, install the project, and run integration tests against nightly and weekly Cisco OS images.
* Wrote a [git hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) that would run RuboCop, a Ruby static analysis tool, against any changed files before running `git commit`.

## rXinu

[Read more about rXinu here](/rxinu/)

## [phil-opp/blog_os](https://github.com/phil-opp/blog_os)

>This blog series creates a small operating system in the Rust programming language. Each post is a small tutorial and includes all needed code, so you can follow along if you like.

I am both a contributor and a user of this repository.  I followed the blog series to create a kernel with memory allocation and interrupt handling. This minimal kernel formed the foundation of rXinu.

In order to give back to a project so integral to the development of `rXinu`, I have submitted numerous patches and helped resolve issues.

## [rust-osdev/multiboot2-elf64](https://github.com/rust-osdev/multiboot2-elf64)

A Multiboot2 library written in Rust.  This crate is generally used in Rust kernels to read basic information passed by a multiboot2 compliant bootloader.

As a collaborator, I assist in maintaining this project and ensuring the latest nightly Rust compiler is supported.

## [phil-opp/linked-list-allocator](https://github.com/phil-opp/linked-list-allocator)

>Simple allocator usable for no_std systems. It builds a linked list from the freed blocks and thus needs no additional data structures.

This memory allocator is used in rXinu as a simple allocator that can be setup with minimal overhead.  As a contributor, I have [submitted an enhancement](https://github.com/phil-opp/linked-list-allocator/pull/7) to improve the handling of an Out Of Memory kernel panic.

## [javar](https://github.com/robert-w-gries/javar)

A Java compiler written in Java. This project compiles a significant subset of Java to MIPS assembly format.

### Features
* Parsing
* Type Checking
* Intermediate Representation
* Register Allocation
* Instruction Assembly

## Educational Network Emulator

Collaborated with a team of Marqutte students to develop an Opendaylight Software Defined Networking (SDN) application that simulated traffic and network conditions, such as latency and traffic duplication.

Our team won second place in the [SDN Innovation Challenge](https://www.extremenetworks.com/solutions/sdn/sdn-innovation-challenge/), an international competion hosted by Extreme Networks and judged by a panel of industry experts.

![image](/assets/images/sdn_innovation_prize.jpg)
