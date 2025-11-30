---
layout: post
title:  "Linux Engineering Mini-Projects"
date:   2025-11-27
categories: linux
---
This post is about Linux things beyond basic administration.

<h2>Bulding Custom Kernel for Kubernetes Workloads</h2>

In this section we will build a custom Kernel tuned for Kubernetes workloads.

First we are going to prepare the environment for compiling the Kernel. You can find the minimal requirements here: [Minimal requirements to compile the Kernel][kernel_requirements]
Since some of the packages should be already present in the Linux system, so, for quick installing, we can run this which should be enough for building the Kernel:
{% highlight ruby %}
sudo apt install -y gcc clang rustc bindgen make bash flex bison pahole mount quota iptables openssl bc tar python3 gawk build-essential libelf-dev libdw-dev elfutils libdwarf-dev zlib1g-dev libssl-dev
{% endhighlight %}

Then, we need to download the Kernel source code (for this example we are going to use v6.x version), which can be found here: [Kernel Source Code][kernel_source]

Once the Kernel source code is downloaded and ready, we can start playing around with command line and parameters by entering the configuration:
{% highlight ruby %}
make menuconfig
{% endhighlight %}
This will open a configuration window for setting various parameters (you can use forward-slash (/) to quickly search for parameters).

Even though using this configuration window is easy since it's graphical, a more efficient approach would be to use it's config file directly, which is called `.config`.

Before doing any edits we are need to know what parameters to change, we could of course experiment and so on, but in keeping this section shorter, we will pretend that we already know what settings we want to change. Since we will be building this Kernel for Kubernetes workloads, then we will have to tick these parameters:
{% highlight ruby %}
# Namespaces
CONFIG_NAMESPACES=y
CONFIG_UTS_NS=y
CONFIG_IPC_NS=y
CONFIG_USER_NS=y
CONFIG_PID_NS=y
CONFIG_NET_NS=y

# Cgroups (v2 preffered)
CONFIG_CGROUPS=y
CONFIG_CGROUP_SCHED=y
CONFIG_CPUSETS=y
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_BPF=y
CONFIG_MEMCG=y
CONFIG_BLK_CGROUP=y

# Containers Filesystems
CONFIG_OVERLAY_FS=y
CONFIG_EXT4_FS=y
CONFIG_XFS_FS=y

# Networking
CONFIG_BRIDGE=y
CONFIG_BRIDGE_NETFILTER=y
CONFIG_NETFILTER=y
CONFIG_NETFILTER_ADVANCED=y
CONFIG_NETFILTER_XT_MATCH_CONNTRACK=y
CONFIG_NETFILTER_XT_MARK=y
CONFIG_NETFILTER_XT_TARGET_MASQUERADE=y
CONFIG_NF_CONNTRACK=y
CONFIG_VXLAN=y
CONFIG_GENEVE=y
CONFIG_BPF=y
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
CONFIG_CGROUP_BPF=y
CONFIG_BPF_EVENTS=y
CONFIG_BPF_LSM=y

# Security
CONFIG_SECCOMP=y
CONFIG_SECCOMP_FILTER=y
CONFIG_SECURITY_YAMA=y
{% endhighlight %}

Now, we will proceed to disable lot of uneeded things:
{% highlight ruby %}
# Filesystems
# CONFIG_EXT2_FS is not set
# CONFIG_EXT3_FS is not set
# CONFIG_JFS_FS is not set
# CONFIG_UFS_FS is not set
# CONFIG_F2FS_FS is not set
# CONFIG_AFS_FS is not set
# CONFIG_NFS_FS is not set
# CONFIG_9P_FS is not set
# CONFIG_CEPH_FS is not set
# CONFIG_OCFS2_FS is not set
# CONFIG_BTRFS_FS is not set

# Networking
# CONFIG_ATALK is not set
# CONFIG_CAN is not set
# CONFIG_BT is not set

# Multi-media
# CONFIG_SOUND is not set
# CONFIG_MEDIA_SUPPORT is not set
# CONFIG_VIDEO_DEV is not set
# CONFIG_DVB_CORE is not set

# Input devices
# CONFIG_INPUT_JOYSTICK is not set
# CONFIG_INPUT_TABLET is not set
# CONFIG_INPUT_TOUCHSCREEN is not set
# CONFIG_INPUT_MISC is not set
# CONFIG_DRM is not set
{% endhighlight %}

Also, since I will be running this in virtual machine, I'm gonna check couple more settings:
{% highlight ruby %}
# KVM, VMware, VirtualBox support
CONFIG_KVM_GUEST=y
CONFIG_PARAVIRT=y
CONFIG_PARAVIRT_CLOCK=y
CONFIG_PARAVIRT_SPINLOCKS=y
CONFIG_VIRTIO=y
CONFIG_VIRTIO_BLK=y
CONFIG_VMWARE_PVSCSI=y
CONFIG_VMWARE_VMCI=y
CONFIG_VBOXGUEST=y
{% endhighlight %}

Now, everything has been configured, we can proceed with building the Kernel, and we can do that with `make` command (for faster build it's possible to use more than one core):
{% highlight ruby %}
# nproc - command to find cores in your computer, I will use 2 cores since my testing computer 4, and I still want to use it at least kinda efficiently
make -j 2
make modules
{% endhighlight %}

Now what we built the image, we can proceed with putting this custom Kernel into a distro, for that we will use Debian. Minimal Debian image can be found here: [Debian][debian]

Next, we will mount that Debian image:
{% highlight ruby %}
sudo apt install libguestfs-tools
# -m /dev/sda1 - mount point on the image (not the host)
sudo guestmount -a debian-12-genericcloud-amd64.qcow2 -m /dev/sda1 /mnt/deb
{% endhighlight %}

<h2>Kernel Tuning</h2>

Starting with system tuning, one thing to remember is that basically everything is already tuned to what is best by default, but that doesn't mean we can't tune the system for specific workloads - do a trade-off. For this we are going to use sysctl.

<h4>Tuning for Network Services (WEB/API)</h4>

The goal is to maximize the bandwidth by adjusting TCP buffer size, connection rate and congestion.

Kernel parameters for maximising TCP buffer size:<br>
- <pre><span style="color: grey;"># absolute hard-limit for read/download</span><br>net.core.rmem_max</pre>
- <pre><span style="color: grey;"># absolute hard-limit for write/upload</span>net.core.wmem_max</pre>
- <pre><span style="color: grey;"># maximum value for TCP read/download</span>net.ipv4.tcp_rmem</pre>
- <pre><span style="color: grey;"># maximum value for TCP write/upload</span>net.ipv4.tcp_wmem</pre>

Kernel parameters for maximising connection rate:<br>
- <pre><span style="color: grey;"># maximum length of the queue of pending connections</span><br>net.core.somaxconn</pre>
- <pre><span style="color: grey;"># maximum length of the queue of packets</span>net.core.netdev_max_backlog</pre>
- <pre><span style="color: grey;"># maximum number of sockets</span>net.ipv4.tcp_max_tw_buckets</pre>

Example configuration:
{% highlight ruby %}
# --- /etc/sysctl.d/99-sysctl.conf

# --- Global Limits
net.core.rmem_max = 33554432
net.core.wmem_max = 33554432
# --- TCP Limits --- Minimum - Default - Maximum
# 4KB minimum, 85KB and 64KB default, 32MB maximum
net.ipv4.tcp_rmem =  4096      87380     33554432
net.ipv4.tcp_wmem =  4096      65536     33554432
{% endhighlight %}


Do not forget to apply these settings:
<pre>sysctl -p</pre>.




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


[kernel_requirements]: https://docs.kernel.org/process/changes.html
[kernel_source]: https://www.kernel.org/pub/linux/kernel/v6.x/
[debian]: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-13.2.0-amd64-netinst.iso
