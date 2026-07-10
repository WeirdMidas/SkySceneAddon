# SkyScene Add-on

**A memory management optimization module for legacy or modern Android platforms**

## Overview

A memory management optimization module for legacy and modern devices on Android platforms. It contains methods and formulas for optimizing memory management for every aspect of the kernel, whether it's managing swapping or data compression in ZRAM for both legacy devices using LRU and cgroup v1 and modern devices using MGLRU and cgroup v2

Compatible with all LMKs. This project will handle only kernel memory management, such as swapping, reclaim and a portion of the userspace, such as managing cached process queues and others, to resolve common problems like thrashing and stalls under heavy usage due to the poor process queue management that traditionally occurs in certain devices

Details see [the lead project](https://github.com/yc9559/qti-mem-opt/commits/master/) & [SkyScene Add-on commits](https://github.com/WeirdMidas/SkySceneAddon/commits/master/)

--------
> ⚠️ **NOTE**  
> On very new devices or those with KSU, SkyScene may cause a bootloop even after uninstallation. Check the requirements and uninstallation methods to avoid further problems if you want to uninstall it
--------

## Main Features

- **📂 Pure optimization, no placebo** - Pure memory management optimization module, not containing other placebo and supporting all mainstream platforms like Qualcomm, MediaTek, Unisoc, and many other platforms
- **🛠️ Adjust the number of background apps to suit your hardware capabilities** - Solve the problem that the background can't hang on devices with more RAM even if the free memory is large, by modifying Android specified ActivityManager `CUR_MAX_CACHED_PROCESSES` based on the RAM tiers of each device individually
- **🥼 Prevent your most important app from being killed** - Customizable list of protected APPs, preventing them from being killed by both in-kernel and in-userspace lowmemorykiller
- **⚠️ Prevent important data from being swapped** - Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations
- **🦺 Allow the user to configure memory recycling threads as they see fit!** - Allow the user to configure the scheduling of memory recycling threads as desired, benefiting from changing their nice, uclamp, cpuset, and shares for these threads, giving maximum freedom for user customization
- **💪 Use Google's recommendations for all supported devices** - Simply apply all the parameters that Google considers "gold" according to your generation, allowing all devices to operate with the correct recommendations for your device
- **🔄 Improve swapping behavior according to the algorithm used** - The way the module optimizes swapping behavior depends entirely on the algorithm used by the kernel, as specified below:
  - **LRU**: Focus on being aggressive so LRU stops being STUPID and IDIOTIC! He needs to learn that we can't leave everything to the last minute and that every SOC has its limits, so make him aggressive to mitigate his naiveté
    - Avoid swapping memory pages which are hard to compress to ZRAM, make the compression rate close to the ideal value of 2.8x for legacy Android
    - Reduce stucks under high memory pressure, reduce the probability of direct memory allocation through higher `watermark_low`, allowing the LRU to respond faster to immediate allocation spikes and atomic allocations
    - Make the LRU less indecisive by considering the limitations and needs of each device individually; avoid a "universal value" for everyone to improve the stupidity of LRU and make it more usable
  - **MGLRU**: Focus on zero swapping costs, free up cycles for user and UX tasks, making memory management completely asynchronous and lightweight on the CPU
    - Scale swap needs based on the device using Google and Samsung's recommendations in this regard, this brings the compression rate closer to the ideal 3.1x for modern Android
- **📝 Let the user customize it a bit!** - Customizable ZRAM size and algorithm, swapfile size and the I/O scheduler. Of course, kernel compatibility is required for all of this
- **📊 Safe, efficient, and tested multiple times** - SELinux can still be enabled

## Requirements

- Android 8 or higher
- A non-unstable ROM or custom kernel
- Magisk (or another root manager, but be aware that it may not be as compatible)
- Having ZRAM enabled in the kernel
- 3GB or more of RAM (Optional)
- Have busybox installed (Optional)

## Installation and Technical Explanations

- Install this module, restart your phone, and open `/sdcard/Android/panel_memcfg.txt` after rebooting to modify the desired params. This will take effect after rebooting
- Open `/sdcard/Android/panel_adjshield.txt` to add the package names of the applications you want to keep running in the background. This will take effect after rebooting
- The ZRAM/Swap size used is as follows:
  - 1GB RAM: 512MB of ZRAM for non-Go devices, 768MB of ZRAM for Go devices
  - 2GB RAM: 1GB of ZRAM for non-Go devices, 1.5GB of ZRAM for Go devices
  - 3GB RAM: 1.5GB ZRAM
  - 4GB RAM: 2GB ZRAM
  - 6GB RAM: 3GB ZRAM
  - 8GB RAM: 4GB ZRAM
  - 12GB or more RAM: 6GB ZRAM
  - Never set ZRAM larger than your physical RAM. To maintain system stability and performance, it is best to cap ZRAM at 75% of your total RAM. Exceeding these limits can lead to high CPU usage (due to constant compression) and a sluggish user experience under heavy loads
    - For older 32-bit devices, do not exceed 4GB of ZRAM. Doing so triggers a "32-bit overflow" error, where the system miscalculates memory, leading to crashes or random reboots
  - The default ZRAM algorithm is lz4, as it is the "gold standard" of the modern Android industry. It compresses more than lzo-rle, is faster than zstd, and has the fastest decompression speed compared to other algorithms
  - ZRAM deduplication is disabled by default because it is computationally expensive and inefficient nowadays, recovering only 5% of space at most, since duplicate data is extremely rare today
- ZSWAP is currently not supported

## Uninstallation instructions and warnings

- Uninstalling this module may cause bootloops on newer devices. To minimize risk, use TWRP for removal. Note that kernels with KernelSU (KSU) often struggle with this module, potentially causing installation failures, high CPU usage, or bugs due to poor kernel implementation

## Documentation and Reference Guide

### Sources

- [ZRAM Tuning Guide](https://juejin.cn/post/7147284908367413261)
- [Cached and Phantom Process](https://github.com/agnostic-apollo/Android-Docs/blob/master/en/docs/apps/processes/phantom-cached-and-empty-processes.md)

### Suggestions for Complementary Modules

- [NoSwipeToKill](https://github.com/dantmnf/NoSwipeToKill): lsposed module by [dantmnf](https://github.com/dantmnf) to reduce the aggressiveness of HyperOS/MIUI in killing processes, recommended for users of Xiaomi ROMs that use lsposed
- [A1Memory](https://github.com/OneB1ank/A1Memory): A memory management module that implements an OEM-style memory management framework in the system server, allowing you to control certain parts of memory management more efficiently than modern Android. Exclusive to Android 8-14

### Usage FAQ

Q: What is this? Is it a one-click full optimization?    
A: It's a Magisk module that optimizes overall memory management. It doesn't include optimizations for I/O, CPU, GPU, Scheduler, or other factors, only memory and its management. This allows the module to make the Android system and kernel behavior more efficient in this regard, favoring better multitasking overall

Q: Can my device use this?    
A: This module is compatible with Android 32/64-bit platforms with Android version >= 8.0, and is not limited to Qualcomm platforms

Q: Does this module become useless if ZRAM is not enabled?    
A: ZRAM control is only a small part of this module's functionality. Using this module without enabling ZRAM will still improve smoothness in low-memory situations. The `unsupported` display in the ZRAM section of the panel file only indicates that the kernel does not support ZRAM; other parameter modifications and cache control are still effective

Q: My device has 12GB or 16GB of physical memory. Do I still need this module?   
A: On some devices, the various device manufacturers (such as Motorola, Xiaomi, Samsung, etc.) have strict limitations on the number of cached processes. Even with ample available memory, background caches may be cleared. This module prevents the rapid clearing of background cache processes, ensuring full utilization of large memory

Q: Why is ZRAM not enabled even after setting the ZRAM size in the configuration file?   
A: This module cannot add ZRAM if the kernel lacks it. Most official kernels support ZRAM, while third-party kernels are less likely to

Q: Is 100% SWAP usage a problem?   
A: No, it's not a problem; rather, it's the expected result. ZRAM is an implementation of swap partitioning, compressing and storing infrequently used memory in pages. The compression ratio is typically 2.8x (on legacy devices), meaning a 2.8GB ZRAM size occupies 1GB of physical memory, equivalent to an additional 1.8GB of RAM. The impact of ZRAM on performance and power consumption depends on the ZRAM size you choose to enable. A larger ZRAM size increases the probability of CPU decompression and reading, and allows more processes to be cached in the background simultaneously. System smoothness is not directly related to swap usage; it depends on the difficulty of memory reclamation and page cache hit rate

Q: Why do background apps still crash?   
A: Physical memory resources are limited and cannot meet unlimited background caching needs. Some vendors may have implemented additional background cache management, such as using LSTM prediction to select and clean up the least likely apps to be used next. If you need to protect certain apps from LMK (userspace or kernel) reclamation, you can add the package names to be protected in the `AdjShield` configuration file. Adding too many apps is not recommended to avoid difficulties in memory reclamation

Q: Why is power consumption increasing?    
A: The caching process itself does not increase power consumption; the increase in power consumption from more page swapping is very limited. It's important to note that while keeping more background apps active, not all of these apps are in a cached dormant state; many services may be running in the background consuming power

### Technical Q&A

Q: What is the limit `CUR_MAX_CACHED_PROCESSES`?    
A: Sometimes, processes aren't killed due to insufficient memory. Checking logcat reveals a limit on the number of cached processes; if the number is >= 31, they are reclaimed. This limit comes from Android's ActivityManager's `max_cached_processes`. The default value for this parameter is stored in `device_config` on newer devices (Android 10 or higher), and in `global settings` on older devices (Android 9 or lower). We use `cmd` to manipulate these values ​​during runtime, allowing the user to benefit from these optimizations at runtime

Q: What are the benefits of pinning frequently used system files to a file page cache?    
A: Google's [Android Performance Tuning Documentation](https://source.android.com/devices/tech/debug/jank_jitter#page-cache) mentions that page cache thrashing is a major cause of long stutters under low memory conditions. In practice, this manifests as a pause of over 100ms when returning to the home screen, especially noticeable in the home screen gesture animation. Generally, the Android framework's `PinnerService` already fixes frequently used files in memory, but some platforms, such as the OnePlus 7 Pro, do not have this service, or the coverage of the fixed memory is insufficient, leading to critical page cache thrashing. This module fixes most commonly used system files in the file page cache, compensating for the shortcomings of existing settings.

Q: How does it prevent specific apps from being cleared by the Android kernel-level LMK?    
A: Even with increasing the LMK trigger threshold and increasing the SWAP space, some critical apps may still not be able to survive in the background indefinitely, let alone keep large games and frequently used chat software running simultaneously on devices with less than 4GB of physical memory. Previous solutions required the Xposed framework to keep a specific app alive, but some people (like me) disliked the Xposed framework. The Android system framework itself, or by notifying the user-space LMKD, calls the `procfs` interface to change the app's `oom_score_adj`. The kernel-space LMK intervenes and terminates the process with the highest `oom_score_adj` when page caching is insufficient. This module's `AdjShield` periodically iterates through `procfs` to match the package name of the app to be protected, intercepting write operations to its `oom_score_adj`, ensuring that the app to be protected is not the first to be terminated by the kernel-space LMK (including simpleLMK and user-space LMK). The `oom_score_adj` of the protected app is fixed at 0. The periodic iteration interval is set to 2 minutes, and the time taken for each iteration is optimized to within 40ms (Cortex-A55@0.8G), causing almost no additional burden on battery life and performance.

Q: What is the relationship between ZRAM and Swap?  
A: ZRAM is an implementation of swap partitioning. When the kernel reclaims memory, inactive anonymous memory pages are swapped into a block device, called swap. This block device can be a separate swap partition, a swapfile, or ZRAM. ZRAM compresses the swapped pages before placing them into memory, resulting in several orders of magnitude lower read/write latency compared to traditional swap methods, leading to better performance

Q: Why not use a swapfile by default?    
A: Swapfiles stored on external storage such as flash memory or hard disks have read/write latency several orders of magnitude higher than ZRAM, which would significantly reduce smoothness, so they are not used. The swapfile option still exists for users who, in turn, have limited memory but want to sacrifice fluidity for slightly better multitasking, even knowing the risks such as I/O degradation, etc

## Credit

@Doug Hoyte  
@topjohnwu  
@卖火炬的小菇凉--改进在红米K20pro上的zram兼容性  
@钉宫--模块配置文件放到更容易找到的位置  
@予你长情i--发现与蝰蛇杜比音效共存版magisk模块冲突导致panel读写错误  
@choksta --协助诊断v4版FSCC固定过多文件到内存  
@yishisanren --协助诊断v5版FSCC在三星平台固定过多文件到内存  
@方块白菜 --协助调试在联发科X10平台ZRAM相关功能  
@Simple9 --协助诊断在Magisk低于19.0的不兼容问题  
@〇MH1031 --协助诊断位于/system/bin二进制工具集的不兼容问题  
@yc9559 -- The creator of qti-mem-opt     
@unintellectual-hypothesis -- Some functions and ideas     
@Iamlooper -- For the magisk MMT Reborn template      