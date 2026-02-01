# SkyScene Add-on

## Modern, LMK-Independent Memory Management

Modern memory management optimization for both legacy and modern devices. Contains strategies to adjust memory behavior for different types and needs between older and modern phones.

## Features

- Pure optimization of modern memory management. Contains solutions and optimizations designed primarily to cover all kernel memory management areas, such as those with traditional or modern swap behavior
  - Compatibility with all LMK types. SkyScene currently does DOES NOT INCLUDE: ART and other optimizations, the only LMK optimization we will perform is in minfree and execution interval, so that the reclaim/process killing part is less impactful for the user (spending less time on reclaim and more time on swapping, which is much cheaper). This means we'll optimize all kernel memory management, such as swapping, compaction, reclaim, mlocked, and it attempts to resolve issues with memory fragmentation, which can impair long-term multitasking, and thrashing from excessive data exchange between ZRAM and RAM. MAYBE including userspace optimizations, but that's not a guarantee
  - Ensure that various areas of the system are using the most up-to-date format according to Google's guidelines. For example, mlocked follows the CTS standard initially available in Android 14 and some Zygote functions recommended by Google. By implementing these "system health" improvements, which may not be present in all ROMs, the device performs slightly better in terms of managing resources
  - As mentioned, the module is a fork of qti-mem-opt, which initially focused on devices with Snapdragon processors but has changed over time. I myself own a device with a Snapdragon processor (a Moto G52 with an SDM680, a terrible processor), so there may be MANY optimizations that only Qualcomm devices can benefit from. But don't be sad, other processors can also benefit from the optimizations we apply here; only Qualcomm devices will receive "special treatment."
- Solve the problem that the background can't hang even if the free memory is large, by adjusting the activity manager, the module increases the amount of background apps for the following:
  - **1GB/2GB**: Instead of **24** apps in the background, it is increased to **32**. In fact, there may be up to **24** phantom processes instead of the original **16**
  - **3GB/4GB**: Instead of **32** apps in the background, it is increased to **64**. In fact, there may be up to **42** phantom processes instead of the original **32**.
  - **6GB**: Instead of **32** apps in the background, it is increased to **96**. In fact, there may be up to **72** phantom processes instead of the original **32**
  - **8GB** or more: Instead of **64** apps in the background, it is increased to **128**. In fact, there may be up to **96** phantom processes instead of the original **64**
  - On Snapdragon devices running Android 9 or lower, changes to the number of background processes are only made within the QTI Framework via system properties. Snapdragon processors with versions higher than this should use the standard activity manager, the same as all other devices
- Customizable list of protected APPs, preventing them from being killed by in-kernel and in-userspace lowmemory killers via adjshield. The user can customize this list of apps as they wish, so be aware of this
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations. However, instead of following Matt Yang's method or even the generic Google Pixel method, a PinnerService approach is used that respects whether the device has MGLRU or LRU, whether the device has a Snapdragon processor, and various other checks such as whether the processor is powerful enough to handle the constant loading of certain libraries. Based on this, the SkyScene Add-on's PinnerService becomes capable of fully adapting to each device, reducing the need for mlocked memory usage as the device allows
- Aim for predictable memory management behavior, preferring to avoid OOM (Out of Memory) in more predictable ways rather than abruptly wiping everything out or restarting the device unexpectedly. It's better to avoid future problems than to suffer over annoying ones
- Make the I/O experience more user-friendly; choose the best algorithm between cfq, maple, none, mq-deadline or ssg (for Qualcomm processors, if available) for UFS storage based on which one they have first, but allow the user to change it whenever they want in the panel. Including something good: allow LRU and MGLRU to benefit from UFS's ability to work consistently to clean up dirty data more predictably, allowing the system to have clean, less cluttered pages in memory. Even on devices without UFS, such as older ones with EMMC, we will still seek to improve the usability and health of the eMMC so that devices using it don't suffer from its excessively poor performance
- Align key maintenance tasks (statistics, cp interval, and dirty data expiration) into a single event. This improves energy efficiency and system predictability, reduces the impact of intermittent wakeups, and enables the CPU to maintain lower, deeper C-states
- Slightly reduce continuous recycling and jitter during high memory usage, align the minfree and execution interval of both LMK and LMKD with the desired behavior, clear memory only when necessary, not immediately for any hiccups that kswapd can handle efficiently, reducing the need for LMK intervention and less intentional "direct reclaim". This benefits both MGLRU and LRU because they spend less time on "reclaim" and can focus solely on swapping with more predictability
- According to the swapping algorithm coupled to the device kernel, there will be differences in the optimization style and techniques used for each, prioritizing covering their weaknesses and advantages in diverse areas without so many drawbacks:
  - **LRU**: Aim for the most aggressive balance between memory compression and cold data swapper. Even though it's a "brute force" strategy, it's necessary because the way LRU thinks is dumb and stupid, forcing it to be aggressive to avoid bigger problems
    - Utilize additional algorithms that can reduce the stupidity of the LRU, allowing the LRU to make minimally decent decisions, making it less troublesome for older kernels
  - **MGLRU**: Aim for maximum efficiency and zero swapping costs, avoid spending more CPU on swapping tasks than strictly necessary, allowing greatly reduces CPU and I/O stalls. So don't let it act blindly; at least make MGLRU capable of reacting to atomic allocations and/or changes in free memory without needing to resort to quickly evicting data
    - Scale the need for swapping using Google and Samsung's standards, which are said to be among the best, as seen in Samsung devices with decent multitasking that doesn't push the SOC to its limits. Furthermore, scale the amount of swapping the system can tolerate depending on the amount of memory, reducing CPU spikes. But as a main strategy: make swapping cheaper than it already is, allowing MGLRU to be extracted to its maximum potential according to kernel compatibility
    - The difference between MGLRU and LRU is noticeable in how they handle margins or "cushions." Instead of using a watermark scale factor (wsf) of 30, as in LRU, to reduce unnecessary swapping, use a scalable wsf based on the amount of memory. This allows MGLRU to adapt correctly to the memory management demands of each device. This reduces unnecessary swapping, CPU usage, and improves MGLRU precision, allowing it to work with maximum respect for the user. It also reduces fragmentation, direct allocation, and high-order allocations in an extremely granular way, respecting the capabilities of each individual hardware component
  - **Both**: Aim for predictable and consistent swapping behavior; avoid "bursty" or system-heavy behavior. With this behavior, swapping on most devices will run with maximum efficiency under load, although it still has its limits, obviously
- It prohibits recycling threads from running on prime cores, running only on big and LITTLE cores, preventing the scheduler from improperly placing these threads on prime cores, wasting energy where these threads could run on big or LITTLE more efficiently, nullifying the impact on energy consumption due to poor scheduling
- Since "fluidity" is essential in the Android system, it becomes necessary to find a way to deal with memory fragmentation WITHOUT proactively forcing compaction or at a higher threshold. Therefore, we will prefer stable memory management that, in turn, tries to prevent fragmentation instead of "curing" it. This is because there are times when it is IMPOSSIBLE to cure memory fragmentation regardless of optimizations, so the best approach is prevention so that multitasking lasts at least a day. Even though this drastically depends on how each device uses memory, it is not a guarantee or a promise; it is an attempt to fix a known problem in Android
- Customizable ZRAM size and algorithm, as well as customizable I/O algorithm, swapfile size, and additional features like the ability to use dedup, choice of how aggressive memory compression is at the expense of speed the algorithms will have (between zstd, deflate and lz4hc), in addition to selecting and recommending a compression algorithm based on SOC capabilities. Of course, kernel compatibility is required for all of this. Also, aim for a compression ratio of 3.1x which is ideal for Modern Android, try to avoid dropping below that
- SELinux can still be enabled

## Requirement

- Compatible with ARM64 and standard ARM
- Magisk, KSU or Apatch, the most up-to-date version possible if you can
- Android 8 or higher. Not compatible with versions below 8
- You need at least 3GB of RAM to use it. Modern Android requires this as a minimum amount of RAM. However, it is still compatible with devices that have less RAM
- ZRAM compatibility is required for most features to work
- It is recommended to use an additional busybox module for situations where the module cannot use the busybox from Magisk or the ROM
- Depending on the ROM or kernel you are using on your device, you may experience compatibility issues if they are heavily modified

## Installation

- Install this module, restart your phone, and open `/sdcard/Android/panel_memcfg.txt` after rebooting to modify the desired ZRAM size, compression algorithm and others params. This will take effect after rebooting.
- Open `/sdcard/Android/panel_adjshield.txt` to add the package names of the applications you want to keep running in the background. This will take effect after rebooting.
- The default SWAP (ZRAM & Swapfile) sizes and parameters are based on AOSP and "serious OEM tuning", which are as follows:
  - 1GB RAM: 512MB ZRAM. A 1GB swapfile is added to supplement the backend.
  - 2GB RAM: 1GB ZRAM. A 1GB swapfile is added to supplement the backend.
  - 3GB RAM: 1.5GB ZRAM.
  - 4GB RAM: 2GB ZRAM.
  - 6GB RAM: 3GB ZRAM.
  - 8GB RAM: 4GB ZRAM.
  - 12GB or more RAM: 6GB ZRAM.
  - It is not recommended to set zram size larger than your device’s physical RAM, as it may cause system instability. It is recommended to maintain a margin of UP TO 75% as ZRAM, leaving 25% for free RAM to work with. Above that, penalties such as excessive CPU usage for compression or inefficient SwapCache space may arise, affecting performance under heavy loads, and even if it allows for greater multitasking, it tends to excessively impact the UX.
  - It also includes automatic selection of the compression algorithm, where lz4 is the primary/fallback, choosing between it and lz4kd, lz4hc and zstd based on the following variables:
    - The device supports these algorithms in the kernel.
    - The device has a LITTLE cluster with 2GHz or more (for zstd)
    - The device has a LITTLE cluster with 1.8GHz or more (for lz4hc)
    - The device has 6GB of RAM or less (for lz4kd)
  - Since the algorithm is automatically chosen after installing the module, it means that the user's device best handles the algorithm to "store the maximum number of pages in ZRAM" without excessive CPU penalties and/or costly decompression. In other words, the "best algorithm" that SkyScene selects is chosen to keep the maximum amount of data in ZRAM, not the fastest one for compression/decompression or other options.
  - The panel also contains a recommendation for the algorithm that has the most potential to be the best for the user's environment, taking into account processor capacity, amount of RAM, and algorithm availability.
  - Algorithms such as lzo, lzo-rlt and deflate are not recommended. They can be used by the user, but are not recommended due to their inefficiency and only being useful in specific ROMs (such as lzo and lzo-rlt being useful in HyperOS/MIUI).
  - ZRAM Dedup is enabled by default on devices with MGLRU. Disabled by default for devices with LRU. Dedup is enabled in MGLRU because MGLRU gives dedup the ability to "see" duplicate cold pages, allowing it to target them specifically, which reduces the energy efficiency and CPU usage penalties imposed by this mechanism.
  - Swapfile is ONLY recommended if you suffer a lot from small multitasking and need more memory urgently, but remember: this will affect your storage considerably, so know what you're doing.
  - Existing users may not receive the automation I designed. This is because the panel doesn't update in real time. A recommendation is to delete the panel and let it be respected in the next update. This is only if you want to take advantage of the automation. If you want to keep everything as it is or configure it yourself, taking advantage of automation features like tips for choosing a better algorithm, etc., you can leave it as is.
- If you have the "mq-deadline" I/O algorithm, only use it if you have UFS below version 4.0, because UFS below that version may suffer from read saturation in demanding situations, something that mq-deadline completely resolves in them.
- ZSWAP is currently not supported.
- Support for per-app cgroup. This is added to devices without cgroupv2 and possess MEMCG, allowing devices to have a slightly better cgroup compared to before, which is only for Android 10 and above.
- Devices with 6GB or less are considered "lowmem," while devices with 8GB or more are considered "high perf."
- Some custom ROMs, such as the recent Lineage OS (Android 15-16), are suffering from excessive memory fragmentation after a few uses, requiring a reboot to fix. SkyScene might help, but fragmentation in these ROMs is inevitable, something only the ROM maintainers can resolve.
- Use only 16kb pages (as found on current devices) IF you want more performance in exchange for slightly losing the ability to sustain background processes. It's a good trade-off for users who would like to gain up to 2-5fps in demanding emulators (such as PC emulators)
- We also offer an optimized Per-Process Reclaim solution for users with Snapdragon devices who want a secondary reclaim that now respects their efficiency needs according to the swapping algorithm the device uses.

## Uninstallation

- If you are going to uninstall the module, to minimize the risk of your ROM not uninstalling the module and interrupting the boot process, I recommend uninstalling it via TWRP. This will almost completely minimize any risk of failure.

## FAQ

### Sources

- [ZRAM Tuning and some other things](https://juejin.cn/post/7147284908367413261).
- [Cached and Phantom Process](https://github.com/agnostic-apollo/Android-Docs/blob/master/en/docs/apps/processes/phantom-cached-and-empty-processes.md).
- [Hybrid Swapping Scheme Based On Per-Process Reclaim](https://ieeexplore.ieee.org/document/8478216)
- Some descriptions of what Pixel device users have experienced.
- Ideas and strategies from Matt Yang in his former qti-mem-opt and perfd-opt projects.
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
@Tatsh -- My full thanks to Wait_until_Login for the improved version