# SkyScene Add-on

## Modern, LMK-Independent Memory Management

Modern memory management optimization for both legacy and modern devices. Contains strategies to adjust memory behavior for different types and needs between older and modern phones.

## Features

- Pure optimization of modern memory management. Contains solutions and optimizations designed primarily to cover all kernel memory management areas, such as those with traditional or modern swap behavior
  - Compatibility with all LMK types. SkyScene currently does DOES NOT INCLUDE: ART, GC, LMK, and other optimizations. This means we'll optimize all kernel memory management, such as swapping, compaction, reclaim, mlocked, and it attempts to resolve issues with memory fragmentation, which can impair long-term multitasking, and thrashing from excessive data exchange between ZRAM and RAM. MAYBE including userspace optimizations, but that's not a guarantee
  - Ensure that various areas of the system are using the most up-to-date format according to Google's guidelines. For example, mlocked follows the CTS standard initially available in Android 14. Then, with the synchronization of these modern solutions with current devices, the device performs more up-to-date and better in terms of memory management
- Solve the problem that the background can't hang even if the free memory is large, by adjusting the activity manager, the module increases the amount of background apps for the following:
  - **1GB/2GB**: Instead of **24** apps in the background, it is increased to **32**
  - **3GB/4GB**: Instead of **32** apps in the background, it is increased to **64**
  - **6GB**: Instead of **32** apps in the background, it is increased to **96**
  - **8GB** or more: Instead of **64** apps in the background, it is increased to **128**
- Customizable list of protected APPs, preventing them from being killed by in-kernel and in-userspace lowmemory killers via adjshield. The user can customize this list of apps as they wish, so be aware of this
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations. However, unlike Matt Yang's method, it is aligned with Google Pixel for better updates
- As a precaution, prevent dirty pages from causing noticeable system stalls. Allow more data to remain in memory, and when needed, send it to storage. This reduces the stalls that occur at certain points in the system proportionally
- According to the swapping algorithm coupled to the device kernel, there will be differences in the optimization style and techniques used for each, prioritizing covering their weaknesses and advantages in diverse areas without so many drawbacks:
  - **LRU**: Aim for the most aggressive balance between memory compression and cold data swapper. Even though it's a "brute force" strategy, it's necessary because the way LRU thinks is dumb and stupid, forcing it to be aggressive to avoid bigger problems
    - Utilize additional algorithms that can reduce the stupidity of the LRU, allowing the LRU to make minimally decent decisions
  - **MGLRU**: Aim for maximum efficiency and zero swapping costs, avoid spending more CPU on swapping tasks than strictly necessary, allowing you to drastically reduce CPU usage from memory management as a whole. This greatly reduces CPU and I/O stalls
    - Scale swapping needs according to the amount of memory, allowing lowmem and high perf devices to have their own swapping priority instead of relying on Google's defaults, which are merely simplistic and don't work for all devices
  - Prefer and prioritize asynchronous swapping over synchronous swapping. Do this respecting the logic that swapping should be a background memory recycling task, and cannot take resources from foreground tasks to avoid unnecessary resource competition, which could generate additional stalls that are complex to deal with
    - Make swapping a background task, following the logic that kswapd needs to be as efficient as possible, reducing the impact of powerful algorithms on the CPU, allowing for better swapping efficiency
  - Offer Per-Process Reclaim as an additional Reclaim, where it is triggered in situations where the device's normal Reclaim does not resolve the situation and needs help, and that's where Per-Process Reclaim comes in and helps the standard reclaim, prioritizing the reclaiming of memory from processes with adj 801 or greater and with the precision varying if the device has MGLRU or LRU
- It prohibits recycling threads from running on prime cores, running only on big and LITTLE cores, preventing the scheduler from improperly placing these threads on prime cores, wasting energy where these threads could run on big or LITTLE more efficiently, nullifying the impact on energy consumption due to poor scheduling 
- Compact memory more often, even if the memory allocation was estimated to be due to a low-memory status. This lets us put more data into RAM at the expense of running compation more often. This is a worthy tradeoff, as it reduces memory fragmentation, which is incredibly important for ZRAM
- Customizable ZRAM size and algorithm, as well as customizable I/O algorithm, swapfile size, and additional features like the ability to use dedup, choice of how aggressive memory compression is at the expense of speed the algorithms will have (between zstd, deflate and lz4hc), in addition to selecting and recommending a compression algorithm based on SOC capabilities, this improves the detection of SOC capabilities. Of course, kernel compatibility is required for all of this. Also, aim for a compression ratio of 3.1x which is ideal for Modern Android, try to avoid dropping below that
- SELinux can still be enabled

## Requirement

- Compatible with ARM64 and standard ARM
- Magisk, KSU or Apatch, the most up-to-date version possible if you can
- Android 8 or higher. Not compatible with versions below 8
- You need at least 3GB of RAM to use it. Modern Android requires this as a minimum amount of RAM. However, it is still compatible with devices that have less RAM
- ZRAM compatibility is required for most features to work
- It is recommended to use an additional busybox module for situations where the module cannot use the busybox from Magisk or the ROM

## Installation

- Install this module, restart your phone, and open `/sdcard/Android/panel_memcfg.txt` after rebooting to modify the desired ZRAM size and compression algorithm. This will take effect after rebooting.
- Open `/sdcard/Android/panel_adjshield.txt` to add the package names of the applications you want to keep running in the background. This will take effect after rebooting.
- The default ZRAM sizes are based on AOSP and "serious OEM tuning", which are as follows:
  - 1GB RAM: 512MB ZRAM for LRU and MGLRU.
  - 2GB RAM: 1GB ZRAM for LRU and 1.2GB for MGLRU.
  - 3GB RAM: 1.5GB ZRAM for LRU and 2.1GB for MGLRU.
  - 4GB RAM: 2GB ZRAM for LRU and 3GB for MGLRU.
  - 6GB RAM: 3GB ZRAM for LRU and MGLRU.
  - 8GB RAM: 4GB ZRAM for LRU and MGLRU.
  - 12GB or more RAM: 6GB ZRAM for LRU and MGLRU.
  - By default, the compression algorithm is lz4; however, through automatic selection of ZRAM compression algorithm, is choosing between zstd and lz4 based on two variables:
    1. The device has a powerful SOC (small cluster above 2GHz).
    2. The device has zstd compatibility in the kernel.
  - ZRAM Dedup is enabled by default on devices with MGLRU. Disabled by default for devices with LRU.
  - It is not recommended to set zram size larger than your device’s physical RAM, as it may cause system instability.
- ZSWAP is currently not supported.
- Support for per-app cgroup. This is added to devices without cgroupv2, allowing devices to have a slightly better cgroup compared to before, which is only for Android 10 and above.

## FAQ

### Sources

- [ZRAM Tuning and some other things](https://juejin.cn/post/7147284908367413261).
- [Cached and Phantom Process](https://github.com/agnostic-apollo/Android-Docs/blob/master/en/docs/apps/processes/phantom-cached-and-empty-processes.md).
- Some descriptions of what Pixel device users have experienced.
- Some parameters that Tytydraco explained in his Ktweak on XDA.
- Some reports and flags from AOSP master. Core optimizations are almost always based on AOSP itself, prioritizing modernization and efficiency above all else.
- Tests and more tests with different configurations and in the same tests (multitasking, games, etc.), I spent hours of my days on this.
- And yes, the names of the SkyScene versions are based on the characters from the movie RIO. A movie set in Brazil, my homeland.

### 使用解答

Q: 这是什么，是一键全优化吗？  
A: 这个是改善Android缓存进程管理的Magisk模块，避免过快地清除后台缓存进程并且改进低内存情况下的流畅度，不包含CPU调度优化之类的其他部分。  

Q: 我的设备能够使用这个吗？  
A: 本模块适用于Android版本>=6.0的安卓32/64位平台，不局限于高通平台。  

Q: 不开启ZRAM是不是这个模块就没用了？  
A: ZRAM控制只是本模块的一小部分功能，不开启ZRAM使用本模块也能够改进低内存情况下的流畅度。在panel文件的ZRAM项目显示`unsupported`仅表示内核不支持ZRAM，其他参数修改和缓存控制还是生效的。  

Q: 我的设备有12GB或者16GB物理内存，还需要这个模块吗？  
A: 在某些设备上由于高通平台缓存进程数量限制比较严格，即使可用内存很多也会出现后台缓存被清除，本模块避免过快地清除后台缓存进程使得大内存得到充分利用。  

Q: 为什么在配置文件设置了ZRAM大小之后还是没开启ZRAM？  
A: 如果内核没有ZRAM功能本模块是不能够添加的。大部分官方内核都支持ZRAM，第三方内核不支持ZRAM的情况多一些。  

Q: SWAP使用率100%这没有问题吗？  
A: 没有问题，不如说这是期望的结果。ZRAM是swap分区的一种实现方式，将不常用内存分页压缩存储，压缩率一般在2.8x，也就是说2.8G的ZRAM大小在物理内存占据1GB的空间，等效于多了1.8G的内存。ZRAM对于性能和耗电的影响取决于你选择开启的ZRAM大小，设置的越大需要CPU解压缩读取的概率越高，同时可在后台同时缓存的进程越多。系统流畅度与SWAP使用率没有直接关系，取决于内存回收难度和页面缓存命中率。  

Q: 为什么后台还是会掉？  
A: 物理内存资源是有限的，不可能满足无限的后台缓存需求。某些厂商可能额外做了后台缓存管理，例如利用LSTM预测来选择清理接下来最不可能使用的APP。如果需要保护某些APP使其免于被内核态LMK回收，可以在`AdjShield`配置文件中添加需要保护的包名，不推荐添加过多的APP避免内存回收出现困难。  

Q: 为什么耗电变多了？  
A: 缓存进程本身是不增加耗电的，更多的页面交换增加的耗电十分有限。需要注意的是保活更多后台APP的同时，这些APP并非全都处于缓存休眠的状态，可能有不少服务在后台运行消耗电量。  

### 技术解答

Q: `CUR_MAX_EMPTY_PROCESSES`这一限制是什么？  
A: 有些时候并非因为内存不足导致进程被杀，查看logcat注意到缓存进程居然有数量上限，数量>=31就回收。这个缓存进程数量上限来自于安卓ActivityManager的`max_cached_processes`，它被初始化为`CUR_MAX_EMPTY_PROCESSES`的一半。这个默认值一般存储在系统框架中无法更改，在高通平台可以通过`ro.vendor.qti.sys.fw.bg_apps_limit`来更改此常量。在较老的平台此值存储在系统分区的`build.prop`中，在较新的平台它存储在`perfconfigstore.xml`。此模块借助Magisk的`MagicMount`可以实现对高通平台缓存进程数量上限的更改。  

Q: 将系统常用文件固定在文件页面缓存有什么用？  
A: 在谷歌的[安卓性能调优文档](https://source.android.com/devices/tech/debug/jank_jitter#page-cache)，提到了低内存情况下页面缓存出现颠簸是长卡顿的主要原因，它在实际中表现为返回桌面时出现100ms以上的停顿，在返回桌面手势动画中尤为明显。一般来说安卓框架的`PinnerService`已经把常用文件固定在内存中，但是某些平台的设备比如一加7Pro并没有这一服务，或者已固定在内存的覆盖范围不够大导致仍然出现关键页面缓存颠簸。此模块将绝大多数系统常用文件固定在文件页面缓存，弥补已有设置的不完善之处。  

Q: 防止特定APP被安卓内核态LMK清除是如何做到的？  
A: 即使调高LMK触发阈值以及加大SWAP空间，某些关键APP可能仍然无法一直在后台存活，更不用说在低于4GB物理内存的设备上想要同时保活大型游戏和常用聊天软件了。在以往的解决方法中，需要Xposed框架实现对指定APP的保活，但是Xposed框架某些人并不喜欢(比如我)。安卓系统框架自身或者通知用户态LMKD调用`procfs`接口更改APP的`oom_score_adj`，内核态LMK在页面缓存不足介入终止`oom_score_adj`最高的进程。本模块的`AdjShield`定期遍历`procfs`匹配需要保护的APP包名，拦截对它的`oom_score_adj`的写入操作，确保需要保护的APP不会是内核态LMK(也包含simpleLMK)最先被终止的。受保护的APP的`oom_score_adj`被固定在0。定期遍历的间隔被设置在2分钟，每次遍历的耗时经过优化控制在40ms(Cortex-A55@0.8G)以内，几乎不会给续航和性能造成额外负担。  

Q: ZRAM和swap是什么关系？  
A: ZRAM是swap分区的一种实现方式。在内核回收内存时，将非活动的匿名内存页换入块设备，被称为swap。这个块设备可以是独立的swap分区，可以是swapfile，也可以是ZRAM。ZRAM将换入的页面压缩后放到内存，所以相比传统的swap方式在读写延迟上低几个数量级，性能更好。  

Q: 为什么不使用swapfile？  
A: 存储在闪存或者磁盘这样外置存储的swapfile，读写延迟比ZRAM高几个数量级，这会显著降低流畅度所以不采用。  

Q: 这个跟SimpleLMK哪个好？  
A: 把Magisk模块跟内核模块对比是不合适的，把SimpleLMK跟LMK对比更加合适。SimpleLMK触发在直接内存分配，LMK触发在kswapd回收结束之后文件页面缓存低于阈值。SimpleLMK触发较晚，优点在于可以尽可能利用全部内存存放活动的匿名页和文件页面缓存，缺点在于文件页面缓存可能出现极低值造成比较长的停顿。LMK触发较早，优点在于主动地维持文件页面缓存水平不容易造成较长的停顿，缺点在于容易受缓存水平波动导致误清除后台缓存进程。本模块调整了LMK的执行代价，缓解了LMK容易受缓存水平波动的问题。  

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
@yc9559 -- Obvious credits that I forgot to put, SkyScene only exists because of him, the GOAT of the 2018-2020 modules  
@unintellectual-hypothesis -- by hybrid swap, conf_mi_reclaim and some ZRAM diskzise    
@Iamlooper -- For the magisk MMT Reborn template. Thanks to the template, I was able to replace the old qti-mem-opt template and keep all the features without extra additions! Also now the cpu usage of the module has reduced by 2%, little but useful