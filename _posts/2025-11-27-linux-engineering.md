---
layout: post
title:  "Linux Engineering"
date:   2025-11-27
categories: linux
---
This post is about Linux things beyond basic administration.

<h2>Kernel Tuning</h2>

Starting with system tuning, one thing to remember is that basically everything is already tuned to what is best by default, but that doesn't mean we can't tune the system for specific workloads - do a trade-off. For this we are going to use sysctl.

<h4>Tuning for Network Services (WEB/API)</h4>

The goal is to maximize the bandwidth by adjusting TCP buffer size, connection rate and congestion.

Kernel parameters for maximising TCP buffer size:<br>
- <pre>net.core.rmem_max</pre> - absolute hard-limit for read/download.
- <pre>net.core.wmem_max</pre> - absolute hard-limit for write/upload.
- <pre>net.ipv4.tcp_rmem</pre> - maximum value for TCP read/download.
- <pre>net.ipv4.tcp_wmem</pre> - maximum value for TCP write/upload.

Example configuration in <pre>/etc/sysctl.d/99-sysctl.conf</pre>:<br>
{% highlight ruby %}
# --- Global Limits
net.core.rmem_max = 33554432
net.core.wmem_max = 33554432
# --- TCP Limits --- Minimum - Default - Maximum
# 4KB minimum, 85KB and 64KB default, 32MB maximum
net.ipv4.tcp_rmem =  4096      87380     33554432
net.ipv4.tcp_wmem =  4096      65536     33554432
{% endhighlight %}

Do not forget to apply these settings using <pre>sysctl -p</pre>.

<h4>Tuning for Databases</h4>
The goal is to maximize I/O efficiency.

<h4>Tuning for Containerization</h4>
The goal is to maximize concurrent processes efficiency.

<br>
<h2>Performance Analysis with eBPF Tools</h2>

To quickly recap eBPF, it's a technology that allowed engineers to build programs that run in kernel space. Some of the tools are <b>perf</b>, <b>strace</b> and <b>flamegraphs</b>.

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
