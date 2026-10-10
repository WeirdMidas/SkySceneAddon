# SkyScene Add-on

**A memory management optimization module for Android platforms**

## Main Features

- Pure memory management optimization module, not containing other placebo and supporting all mainstream platforms like Qualcomm, MediaTek, Unisoc, and many other platforms
- Customizable list of protected APPs, preventing them from being killed by both in-kernel and in-userspace lowmemorykiller
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations
- The way the module optimizes swapping behavior depends entirely on the algorithm used by the kernel, as specified below:
  - **Traditional LRU**: Abandon the aggressiveness we wanted before, focus on making Traditional LRU more precise, lighter on the CPU, and avoiding evicting/swapping important pages
    - Make kswapd act smoothly and predictably, without CPU usage spikes to meet user demands. Maintain a healthy compression ratio of 2.8x as much as possible
  - **MGLRU**: Focus on delivering more efficient memory recycling behavior, avoid unnecessary aggression, and always make swapping the first option, leaving LMK as a last resort
    - Scale swapping needs based on the gold standard values ​​recommended by Google and Samsung. This maintains a healthy compression ratio of 3.1x or higher for this algorithm
- The module provides up to 6GB of ZRAM and 4GB of Swapfile, with the ability to select the compression algorithm. Many parameters can be modified in the configuration parameter file, in addition to, of course, needing compatibility with your kernel. They will take effect after reboot. Please pay attention to the comments
- SELinux can still be enabled

## Requirements

- Android 8 or higher
- It is recommended that the module be used in a stock kernel; some custom kernels may have such aggressive memory management that the module could negatively interfere
- Magisk or another root manager
  - KernelSU in poorly implemented kernels may have problems with the module
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

## Documentation and Reference Guide

### Sources

- [ZRAM Tuning Guide](https://juejin.cn/post/7147284908367413261)

### Suggestions for Complementary Modules

- [NoSwipeToKill](https://github.com/dantmnf/NoSwipeToKill): lsposed module by [dantmnf](https://github.com/dantmnf) to reduce the aggressiveness of HyperOS/MIUI in killing processes, recommended for users of Xiaomi ROMs that use lsposed
- [A1Memory](https://github.com/OneB1ank/A1Memory): A memory management module that implements an OEM-style memory management framework in the system server, allowing you to control certain parts of memory management more efficiently than modern Android. Exclusive to Android 8-14

### Usage FAQ

Q: What is this? Is it a one-click full optimization?    
A: It's a Magisk module that optimizes overall memory management. It doesn't include optimizations for CPU, GPU, Scheduler, or other factors, only memory and its management. This allows the module to make the Android system and kernel behavior more efficient in this regard, favoring better multitasking overall

Q: Can my device use this?    
A: This module is compatible with Android 32/64-bit platforms with Android version >= 8.0, and is not limited to Qualcomm platforms

Q: Does this module become useless if ZRAM is not enabled?    
A: ZRAM control is only a small part of this module's functionality. Using this module without enabling ZRAM will still improve smoothness in low-memory situations. The `unsupported` display in the ZRAM section of the panel file only indicates that the kernel does not support ZRAM; other parameter modifications and cache control are still effective

Q: Why is ZRAM not enabled even after setting the ZRAM size in the configuration file?   
A: This module cannot add ZRAM if the kernel lacks it. Most official kernels support ZRAM, while third-party kernels are less likely to

Q: Is 100% SWAP usage a problem?   
A: No, it's not a problem; rather, it's the expected result. ZRAM is an implementation of swap partitioning, compressing and storing infrequently used memory in pages. The compression ratio is typically 2.84x, meaning a 2.84GB ZRAM size occupies 1GB of physical memory, equivalent to an additional 1.84GB of RAM. The impact of ZRAM on performance and power consumption depends on the ZRAM size you choose to enable. A larger ZRAM size increases the probability of CPU decompression and reading, and allows more processes to be cached in the background simultaneously. System smoothness is not directly related to swap usage; it depends on the difficulty of memory reclaim and page cache hit rate

Q: Why do background apps still close?   
A: Physical memory resources are limited and cannot meet unlimited background caching needs. Some vendors may have implemented additional background cache management, such as using LSTM prediction to select and clean up the least likely apps to be used next. If you need to protect certain apps from LMK reclaim, you can add the package names to be protected in the `AdjShield`  configuration file. Adding too many apps is not recommended to avoid difficulties in memory reclaim

Q: Why is power consumption increasing?    
A: The caching process itself does not increase power consumption; the increase in power consumption from more page swapping is very limited. It's important to note that while keeping more background apps active, not all of these apps are in a cached dormant state; many services may be running in the background consuming power

### Technical Q&A

Q: What are the benefits of pinning frequently used system files to a file page cache?    
A: Google's [Android Performance Tuning Documentation](https://source.android.com/devices/tech/debug/jank_jitter#page-cache) mentions that page cache thrashing is a major cause of long stutters under low memory conditions. In practice, this manifests as a pause of over 100ms when returning to the home screen, especially noticeable in the home screen gesture animation. Generally, the Android framework's `PinnerService` already fixes frequently used files in memory, but some platforms, such as the OnePlus 7 Pro, do not have this service, or the coverage of the fixed memory is insufficient, leading to critical page cache thrashing. This module fixes most commonly used system files in the file page cache, compensating for the shortcomings of existing settings

Q: How does it prevent specific apps from being cleared by the LMK?    
A: Even with improvements that make swapping cheaper than resorting to LMK, some critical apps may still not be able to survive in the background indefinitely, let alone keep large games and frequently used chat software running simultaneously on devices with less than 4GB of physical memory. Previous solutions required the Xposed framework to keep a specific app alive, but some people (like me) disliked the Xposed framework. The Android system framework itself, or by notifying the user-space LMKD, calls the `procfs` interface to change the app's `oom_score_adj`. The LMK in general intervenes and terminates the process with the highest `oom_score_adj` when page caching is insufficient. This module's `AdjShield` periodically iterates through `procfs` to match the package name of the app to be protected, intercepting write operations to its `oom_score_adj`, ensuring that the app to be protected is not the first to be terminated by the kernel-space LMK (including simpleLMK and user-space LMK). The `oom_score_adj` of the protected app is fixed at 0. The periodic iteration interval is set to 2 minutes, and the time taken for each iteration is optimized to within 40ms (Cortex-A55@0.8G), causing almost no additional burden on battery life and performance

Q: What is the relationship between ZRAM and Swapfile?  
A: ZRAM is an implementation of swap partitioning. When the kernel reclaims memory, inactive anonymous memory pages are swapped into a block device, called swap. This block device can be a separate swap partition, a swapfile or ZRAM. ZRAM compresses the swapped pages before placing them into memory, resulting in several orders of magnitude lower read/write latency compared to traditional swap methods, leading to better performance

Q: Why not use a swapfile by default?    
A: Swapfiles stored on external storage such as flash memory or hard disks have read/write latency several orders of magnitude higher than ZRAM, which would significantly reduce smoothness, so they are not used. The swapfile option still exists for users who, in turn, have limited memory but want to sacrifice fluidity for more virtual memory (for example: trying to run heavy emulators), even knowing the risks such as I/O degradation, etc

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