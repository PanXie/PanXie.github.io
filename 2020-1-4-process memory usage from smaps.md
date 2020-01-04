---
layout:     post
title:      A process' memory usage from /proc/pid/smaps
subtitle:   
date:       2020-4
author:     XP
header-img: img/problems.jpg
catalog: true
tags:
    - memory
    - /proc/pid/smaps
    - Shared_Clean
    - Shared_Dirty
    - Private_Clean
    - Private_Dirty
---

## A process' memory usage from /proc/pid/smaps ##
----

来源：https://unix.stackexchange.com/questions/33381/getting-information-about-a-process-memory-usage-from-proc-pid-smaps


### 1. Answer ###

Clean pages are pages that have not been modified since they were mapped (typically, text sections from shared libraries are only read from disk (when necessary), never modified, so they'll be in shared, clean pages).
Dirty pages are pages that are not clean (i.e. have been modified).

Private pages are available only to that process, shared pages are mapped by other processes*.

RSS is the total number of pages, shared or not, currently mapped into the process. So `Shared_Clean` + `Shared_Dirty` would be the shared part of the RSS (i.e. the part of RSS that is also mapped into other processes), and `Private_Clean` + `Private_Dirty` the private part of RSS (i.e. only mapped in this process).

PSS (proportional share size) is as you describe. Private pages are summed up as is, and each shared mapping's size is divided by the number of processes that share it.
So if a process had 100k private pages, 500k pages shared with one other process, and 500k shared with four other processes, the PSS would be:

```
100k + (500k / 2) + (500k / 5) = 450k
```


Further readings:

- [ELC: How much memory are applications really using?](https://lwn.net/Articles/230975/)
- [Documentation/filesystems/proc.txt](https://www.kernel.org/doc/Documentation/filesystems/proc.txt) in the kernel source
- [man proc(5)](http://man7.org/linux/man-pages/man5/proc.5.html)
- [Linux Memory Management Overview](http://tldp.org/LDP/khg/HyperNews/get/memory/linuxmm.html)
- [Memory Management](http://tldp.org/LDP/tlk/mm/memory.html) at TLDP.org
- [LinuxMM](https://linux-mm.org/)
  
Regarding process-wide sums:

- RSS can be (approximately+) obtained by summing the Rss: entries in smaps (you don't need to add up the shared/private shared/dirty entries).

```
awk '/Rss:/{ sum += $2 } END { print sum }' /proc/$$/smaps
```

- You can sum up Pss: values the same way, to get process-global PSS.

- USS isn't reported in smaps, but indeed, it is the sum of private mappings, so you can obtain it the same way too


### 2, About VSS，RSS，PSS，USS ###

![VSS/RSS/PSS/USS](../img/VSS_PSS_RSS_USS.png)
