---
layout: post
title:  "Linux Engineering"
date:   2025-11-27
categories: linux
---
This post is about Linux things beyond basic administration.

<h2>Kernel Tuning</h2>

Starting with system tuning, one thing to remember is that basically everything is already tuned to what is best by default, but that doesn't mean we can't tune the system for specific workloads - do a trade-off. For this we are going to use sysctl.

<b>Tuning for Network Services (WEB/API)</b>
The goal is to maximize the bandwidth.

<b>Tuning for Databases</b>
The goal is to maximize I/O efficiency.

<b>Tuning for Containerization</b>
The goal is to maximize concurrent processes efficiency.

<br>
<h2>Performance Analysis with eBPF Tools</h2>

To quickly recap eBPF, it's a technology that allowed engineers to build programs that run in kernel space. Some of the tools are perf, strace and flamegraphs.

<br>
<h2>Memory Management</h2>

<br>
<h2>Process Management</h2>

<br>
<h2>Systemd and Boot Management</h2>

<br>
<h2>Networking</h2>

<br>
<h2>File Systems and Storage Management</h2>


[jekyll]: https://jekyllrb.com
