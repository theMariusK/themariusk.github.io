---
layout: post
title:  "Linux Engineering Mini-Projects"
date:   2025-11-27
categories: linux
---
This post is about Linux things beyond basic administration.

<h2>Bulding and Running Custom Kernel</h2>

In this section we will build and run a custom Kernel, and for more interseting purpose, we are going to tune it for Kubernetes workloads.

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

Before doing any edits we are need to know what parameters to change, we could of course experiment and so on, but in keeping this section simple and short, we are just gonna ask an AI to generate us a sample `.config` file for monolith Kernel (meaning it will have no modules):
{% highlight ruby %}
#
# BASIC SYSTEM & MONOLITHIC SETTINGS
#
CONFIG_64BIT=y
CONFIG_X86_64=y
CONFIG_X86=y
CONFIG_MMU=y
CONFIG_EXPERT=y
CONFIG_EMBEDDED=y

# Monolithic kernel
# CONFIG_MODULES is not set

# Kernel command line + initrd
CONFIG_BLK_DEV_INITRD=y
CONFIG_INITRAMFS_SOURCE=""
CONFIG_RD_GZIP=y
CONFIG_RD_XZ=y
CONFIG_RD_BZIP2=y
CONFIG_RD_LZ4=y

#
# BLOCK + PCI + VIRTIO (required to avoid VFS errors)
#
CONFIG_BLOCK=y
CONFIG_BLK_DEV=y
CONFIG_PARTITION_ADVANCED=y
CONFIG_EFI_PARTITION=y
CONFIG_MSDOS_PARTITION=y

# /dev population – CRITICAL
CONFIG_DEVTMPFS=y
CONFIG_DEVTMPFS_MOUNT=y
CONFIG_TMPFS=y
CONFIG_TMPFS_POSIX_ACL=y
CONFIG_TMPFS_XATTR=y
CONFIG_DEVPTS_MULTIPLE_INSTANCES=y

# PCI + Virtio
CONFIG_PCI=y
CONFIG_PCI_MSI=y
CONFIG_VIRTIO=y
CONFIG_VIRTIO_PCI=y
CONFIG_VIRTIO_BLK=y
CONFIG_VIRTIO_NET=y
CONFIG_VIRTIO_CONSOLE=y
CONFIG_VIRTIO_BALLOON=y
CONFIG_VIRTIO_INPUT=y

#
# FILESYSTEMS
#
CONFIG_EXT4_FS=y
CONFIG_EXT4_FS_POSIX_ACL=y
CONFIG_EXT4_FS_SECURITY=y

# VFAT/FAT for /boot/efi
CONFIG_FAT_FS=y
CONFIG_MSDOS_FS=y
CONFIG_VFAT_FS=y
CONFIG_FAT_DEFAULT_UTF8=y

# REQUIRED NLS (fixes cp437 / codepage errors)
CONFIG_NLS=y
CONFIG_NLS_DEFAULT="utf8"
CONFIG_NLS_CODEPAGE_437=y
CONFIG_NLS_CODEPAGE_850=y
CONFIG_NLS_ASCII=y
CONFIG_NLS_UTF8=y
CONFIG_NLS_ISO8859_1=y
CONFIG_NLS_ISO8859_15=y

# OverlayFS (required for container layers)
CONFIG_OVERLAY_FS=y
CONFIG_OVERLAY_FS_REDIRECT_DIR=y
CONFIG_OVERLAY_FS_INDEX=y
CONFIG_OVERLAY_FS_XINO_AUTO=y

# AutoFS (silences systemd failure)
CONFIG_AUTOFS4_FS=y

#
# NAMESPACES (required for containers)
#
CONFIG_NAMESPACES=y
CONFIG_UTS_NS=y
CONFIG_IPC_NS=y
CONFIG_USER_NS=y
CONFIG_PID_NS=y
CONFIG_NET_NS=y
CONFIG_TIME_NS=y

#
# CGROUPS (cgroup v2)
#
CONFIG_CGROUPS=y
CONFIG_CGROUP_SCHED=y
CONFIG_FAIR_GROUP_SCHED=y
CONFIG_CFS_BANDWIDTH=y
CONFIG_RT_GROUP_SCHED=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CPUSETS=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_PERF=y
CONFIG_BLK_CGROUP=y
CONFIG_MEMCG=y
CONFIG_MEMCG_SWAP=y
CONFIG_MEMCG_KMEM=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_RDMA=y
CONFIG_CGROUP_MISC=y
CONFIG_CGROUP_BPF=y

# CGROUP v2 unified hierarchy support
CONFIG_CGROUP2=y

#
# EBPF / BPF
#
CONFIG_BPF=y
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
CONFIG_BPF_EVENTS=y
CONFIG_HAVE_EBPF_JIT=y
CONFIG_FTRACE=y
CONFIG_KPROBES_ON_FTRACE=y

CONFIG_NET_CLS_BPF=y
CONFIG_NET_ACT_BPF=y
CONFIG_BPF_STREAM_PARSER=y
CONFIG_BPF_LSM=y

#
# NETWORKING (K8s compatible)
#
CONFIG_NET=y
CONFIG_PACKET=y
CONFIG_UNIX=y
CONFIG_INET=y
CONFIG_TCP_CONG_CUBIC=y
CONFIG_DEFAULT_TCP_CONG="cubic"

# IPv6
CONFIG_IPV6=y
CONFIG_IPV6_ROUTER_PREF=y
CONFIG_IPV6_ROUTE_INFO=y
CONFIG_IPV6_MULTIPLE_TABLES=y

# Bridge/veth/tunnels
CONFIG_BRIDGE=y
CONFIG_VETH=y
CONFIG_VLAN_8021Q=y
CONFIG_NET_IP_TUNNEL=y
CONFIG_TUN=y
CONFIG_VXLAN=y
CONFIG_GENEVE=y

# Netfilter (iptables + nftables)
CONFIG_NETFILTER=y
CONFIG_NETFILTER_ADVANCED=y
CONFIG_NF_CONNTRACK=y
CONFIG_NF_NAT=y
CONFIG_NF_NAT_MASQUERADE_IPV4=y
CONFIG_NF_NAT_MASQUERADE_IPV6=y

# iptables
CONFIG_NETFILTER_XTABLES=y
CONFIG_NETFILTER_XT_TARGET_MASQUERADE=y
CONFIG_NETFILTER_XT_MATCH_ADDRTYPE=y
CONFIG_NETFILTER_XT_MATCH_CONNTRACK=y
CONFIG_IP_NF_IPTABLES=y
CONFIG_IP_NF_FILTER=y
CONFIG_IP_NF_NAT=y

# nftables
CONFIG_NF_TABLES=y
CONFIG_NF_TABLES_INET=y
CONFIG_NFT_CT=y
CONFIG_NFT_COUNTER=y
CONFIG_NFT_MASQ=y
CONFIG_NFT_NAT=y

#
# LVM + DM-crypt (safe defaults)
#
CONFIG_MD=y
CONFIG_BLK_DEV_DM=y
CONFIG_DM_CRYPT=y
CONFIG_DM_SNAPSHOT=y
CONFIG_DM_THIN_PROVISIONING=y
CONFIG_DM_MIRROR=y
CONFIG_DM_ZERO=y
CONFIG_DM_VERITY=y

# Crypto primitives for dm-crypt
CONFIG_CRYPTO_AES=y
CONFIG_CRYPTO_XTS=y
CONFIG_CRYPTO_SHA256=y
CONFIG_CRYPTO_SHA512=y
CONFIG_CRYPTO_CBC=y
CONFIG_CRYPTO_ECB=y
CONFIG_CRYPTO_USER_API_HASH=y
CONFIG_CRYPTO_USER_API_SKCIPHER=y

#
# SYSTEMD / USERSpace helpers
#
CONFIG_SIGNALFD=y
CONFIG_TIMERFD=y
CONFIG_EVENTFD=y
CONFIG_EPOLL=y
CONFIG_FHANDLE=y
CONFIG_SHMEM=y
CONFIG_AIO=y
CONFIG_IO_URING=y
CONFIG_INOTIFY_USER=y
CONFIG_FANOTIFY=y

#
# VIRTUALIZATION (guest)
#
CONFIG_PARAVIRT=y
CONFIG_PARAVIRT_CLOCK=y
CONFIG_KVM_GUEST=y
CONFIG_PARAVIRT_SPINLOCKS=y

#
# SERIAL CONSOLE (QEMU)
#
CONFIG_TTY=y
CONFIG_VT=y
CONFIG_VT_CONSOLE=y
CONFIG_SERIAL_8250=y
CONFIG_SERIAL_8250_CONSOLE=y

#
# SCHEDULER / TIMERS
#
CONFIG_NO_HZ_IDLE=y
CONFIG_HIGH_RES_TIMERS=y
CONFIG_PREEMPT_NONE=y
CONFIG_HZ_100=y

#
# MISC
#
CONFIG_PROC_FS=y
CONFIG_SYSFS=y
CONFIG_HUGETLBFS=y
CONFIG_HUGETLB_PAGE=y
CONFIG_SWAP=y
{% endhighlight %}

Also, since we changed `.config` contents with ours, we need to fill in the defaults, it can be done using:
{% highlight ruby %}
make olddefconfig
{% endhighlight %}

Now, everything has been configured, we can proceed with building the Kernel, and we can do that with `make` command (for faster build it's possible to use more than one core):
{% highlight ruby %}
# nproc - command to find cores in your computer, I will use 2 cores since my testing computer 4, and I still want to use it at least kinda efficiently
make -j 2
{% endhighlight %}

Now that we built the image, we can proceed with downloading a Debian image which will be used to run this custom Kernel, it can be found here: [Debian][debian]

Finally, we will use QEMU/KVM to run the image as a virtual machine, for that we will need the tools (`sudo apt install qemu-system-x86`), after all the things was taken care of, I have made a simple script to run our Debian Image with custom Kernel:
{% highlight ruby %}
# run_custom_kernel.sh
#!/bin/bash

[ $# -lt 1 ] && echo "Please provide image path" && exit 1

qemu-system-x86_64 \
	-kernel linux-6.17/arch/x86/boot/bzImage \
	-drive file=$1,if=virtio,format=qcow2 \
	-append "root=/dev/vda1 console=ttyS0" \
	-m 1G \
	-enable-kvm \
	-nographic
{% endhighlight %}

This runs our custom Kernel in the Debian image. Once the OS is loaded, we login with `root` and check the Kernel:
{% highlight ruby %}
uname -r
# 6.17.0
{% endhighlight %}

That's it, we successfully built and ran custom Kernel in a Debian image.

<h2>Kernel Tuning</h2>

Apart from building a custom Kernel, we can as well improve specific functions during runtime. In system tuning, one thing to remember is that basically everything is already tuned to what is best by default, but that doesn't mean we can't tune the system for specific workloads - do a trade-off. For this we are going to use sysctl for a couple of use-cases.

<h4>Tuning for Network Services (WEB/API)</h4>

The goal is to maximize the bandwidth by adjusting TCP buffer size, connection rate and congestion.

Kernel parameters for maximising TCP buffer size:<br>
- <pre><span style="color: grey;"># absolute hard-limit for read/download</span><br>net.core.rmem_max</pre>
- <pre><span style="color: grey;"># absolute hard-limit for write/upload</span><br>net.core.wmem_max</pre>
- <pre><span style="color: grey;"># maximum value for TCP read/download</span><br>net.ipv4.tcp_rmem</pre>
- <pre><span style="color: grey;"># maximum value for TCP write/upload</span><br>net.ipv4.tcp_wmem</pre>

Kernel parameters for maximising connection rate:<br>
- <pre><span style="color: grey;"># maximum length of the queue of pending connections</span><br>net.core.somaxconn</pre>
- <pre><span style="color: grey;"># maximum length of the queue of packets</span><br>net.core.netdev_max_backlog</pre>
- <pre><span style="color: grey;"># maximum number of sockets</span><br>net.ipv4.tcp_max_tw_buckets</pre>

<h4>Tuning for Containerization</h4>
The goal is to maximize concurrent processes efficiency.

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

[kernel_requirements]: https://docs.kernel.org/process/changes.html
[kernel_source]: https://www.kernel.org/pub/linux/kernel/v6.x/
[debian]: https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-nocloud-amd64.qcow2
