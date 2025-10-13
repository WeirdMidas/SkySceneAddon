# SkyScene Add-On
![1000007971](https://github.com/user-attachments/assets/07013c32-24b4-48db-9dbd-c36a49089755)

"The challenge of modernity is to live without illusions, without becoming disillusioned."
## Modern and user-friendly memory management

A modern memory and I/O management module for Android platforms. A complete focus on modernity and efficiency, with nothing outdated, hacked, tricked, or otherwise inconsistencies due to unfounded dev recommendations. SkyScene is based on the idea of ​​this module here: https://developer.aliyun.com/article/1230689.

---

> [RECOMMENDATION!]
> We recommend using the [Basic Cleaner](https://github.com/WeirdMidas/BasicCleaner) If you want a complete organization and cleaning of your system, along with that, the Android Runtime optimizations will be "complete".

---

> [URGENT NOTICE!]
> Recently, users and developers have reported that several modules corrupted their systems after uninstallation. Like SkyScene, the reason for this is that current root managers are changing their code to something more integrated with the kernel to escape Google Play. The problem is that this has ended up generating horrible bugs, with some modules causing MASSIVE syntax errors when installing/uninstalling! So, if you installed SkyScene and would like to uninstall it, I apologize if your device suffers a bootloop; the problem will no longer be my problem, as it is already beyond what my module could cause.

---

## ⭐ Features

- Pure memory and I/O management optimization, it does not contain any other placebo or anything that current Androids no longer use or are deprecated or outdated. Contains only modern optimizations and improvements for recent Android devices, with backward compatibility and optimizations for older devices. Depending on the device's memory availability, follow two different strategies for each:
  - **Lowmem**: Follow the "Free memory is waste, but used unnecessarily is also waste" strategy. Instead of following the "Google-like" behavior, SkyScene aims to make lowmem devices use RAM as efficiently as possible, considering their limitations. They cannot leave unused data in memory; they need space for continuous allocation tasks due to their extreme limitations. Prioritizing multitasking on these devices as much as possible
  - **High Perf**: Follow the "Google-like" strategy, the famous "free memory is waste." Allowing high-perf devices to extract the maximum from their memory capacity, the only thing they need to worry about is their resource allocation capacity. It prioritizes a balance between multitasking and maximum fluidity
  - **Both**: Even without scheduling or CPU/GPU optimizations. Prioritize fluidity over most memory states, improve fluidity in light states, and maintain fluidity up to the multitasking limit. Follow the qti-mem-opt proposal, as a way to honor the legacy of Matt Yang's module
- The module can avoid background interruption by modifying the memory management mechanism (lmk, psi), providing four optimized forms of these mechanisms that the user can choose when installing the module, as follows:
  - **lmkd psi**: Uses the psi form mentioned by Google, making it closer to a 'complete replacement for minfree' as Google tries to integrate it, lmkd psi comes with optimizations that allow it to become more accurate, even on devices with less memory, recognizing thresholds more accurately and respecting the user above all. PSI will work so that it will respect the user's multitasking as much as it can. However, if fluidity is compromised, the PSI will act to resolve the situation as much as possible, always based on whether or not the user's perception of fluidity will be negatively affected
  - **lmkd minfree**: Minfree will work like the classic minfree from old lmk. As said, lmkd minfree will work in a way that it tries to keep the device out of OOM situations as much as possible, it is like a guard that does not let the client get into too much danger
  - **lmkd minfree + psi**: An older implementation used on devices that didn't have the most modern PSI possible. With PSI optimizations in more modern devices, the need for minfree has become obsolete, giving more room and space to pure PSI. However, older devices can still benefit from this combination, which, according to the authors, can bring them closer to near-ideal memory management
  - **old lmk**: For devices with the old lmk via driver, such as devices with Android below 10. They will also receive optimizations that will be designed to keep as much data in memory as possible, before deciding to kill something. Also included is the ability for the user to choose to use the old lmk or lmkd if they are on Android 10+ and still have the old lmk driver
- Different devices prioritize dynamic killing methods based on their device's limitations and human perception (using psychology studies). As mentioned below:
  - **Lowmem**: They prefer to have stall tolerance and confidently trust that reclaiming will solve the problem, even in the worst-case scenarios. In human perception, 200ms-400ms of stalling are perceived as acceptable and noticeable, so lmkd lowmem will work within this range. Relying as much as possible on reclaiming prevents the user experience from becoming extremely annoying and unbearable, such as stalls over 500ms
  - **High perf**: They prefer to have no stall tolerance; their killing method falls within the user's perception of not noticing or noticing stalls. Where high perf devices prefer to kill processes based on whether they have a limit between 100-300ms, and their lmkd will work in that range. Based on this, one might expect high-performance users to prioritize smooth gameplay without any repercussions. After all, high-performance devices typically have to maintain high FPS to please these users
- There are also additional flags that improve the behavior of lmk depending on the version of Android, kernel or ROM, which are:
  - **New Strategy (lmkd + android 13+)**: Uses all metrics equally, prioritizing usability across processes. Based on this, lmkd begins to make decisions with greater accuracy and significantly lower false positives, allowing lmkd to be proactive and adaptive as needed
  - **Page Cache Margin (lmkd)**: The page cache has a margin of 5% of total RAM. In other words, the page cache cannot fall below this value. This allows for better lmkd accuracy depending on the amount of memory and its allocation needs
  - **Per-app cgroup control (For devices with cgroupv1 + lmkd)**: A way to partially integrate the cgroupv2 intelligence of modern devices into older devices. This allows lmkd and memory recycling threads to have more precision over application memory consumption, improving aspects such as memory compression, etc. This even makes MGLRU and LRU more efficient in this environment
  - **Dynamic Watermark Boost (lmkd psi + lmkd psi & minfree)**: As memory (RAM + ZRAM) becomes saturated, the need for cleanup becomes increasingly urgent to prevent the device from stalling due to complete RAM and ZRAM saturation. Increase the watermark boost in smaller increments as the saturation increases, minimizing saturation as much as possible by killing the least important processes at the time. This improves multitasking sustainability
  - **Reset lmkd after all main optimizations (lmkd)**: After integrating all SkyScene optimizations, such as VM, Cgroup, etc. Reset lmkd so that it records the current state of the system and kernel, allowing it to record all pre-made patterns solved by SkyScene
  - **No-batching (old lmk)**: On OnePlus devices or with the most up-to-date LMK driver, they come built-in with "quick decision", but we don't need that, we have more precision by optimizing the LMK driver with a focus on retaining data than using these mechanisms
  - **Use lmk's oom_reaper instead of the VM's (old lmk)**: Prioritize the use of the lmk driver's oom_reaper over the VM subsystem's, allowing heap memory release to be more efficient and consistent with the behavior of these devices
- Abandon the old hierarchy solution we created. Follow a new proposal for memory management threads. Synchronize all threads, allowing them all to work with the same weight and, in turn, together. Furthermore, improve thread scheduling and ensure that they cannot use prime cores, only small cores (in the case of SOCs with six small cores) or both small and big cores. This eliminates the need for prime cores, avoiding wasted power even on devices with two prime cores, allowing swapping to perform with above-average efficiency
- It also includes optimizations on how many background processes the system can allow. Depending on the amount of memory on the device, the number of cached/background processes can scale, following this style:
  - **2GB** can have up to **32** cached/background apps instead of **24**
  - **3GB/4GB** can have up to **64** cached/background apps instead of **32**
  - **6GB** can have up to **96** cached/background apps instead of **32**
  - **8GB** or more can have up to **128** cached/background apps instead of **64**
- On devices without **MGLRU**, avoid premature reclaim by using watermark_scale_factor consistent with the amount of memory on the device, which makes reclaim less aggressive and more efficient, causing the user to have an increase of up to ~2x in the number of open processes compared to before, and still with the same or slightly lower memory consumption in multitasking than the user used previously, that is, without a negative impact on the amount of memory available in multitasking without being greater than the user normally used
  - For **2-3GB** of RAM, it is set to **1**, due to its memory limitations
  - For **4GB** of RAM, it is set to **20**
  - For **6GB** of RAM, it is set to **25**
  - For **8GB** of RAM, it is set to **30**
  - For **12GB** of RAM or more, it is set to **40**
  - For devices with **MGLRU**, it is set to **1**
- Abandon old swapping solutions such as proactive and reactive swapping. In addition, separates swapping optimizations between LRU (old and less intelligent) and MGLRU (modern and smarter). As swapping behavior does not change according to Android versions, but the kernel. It becomes easier to optimize to the way the two have been made, allowing the LRU to follow an aggressive strategy that avoids CPU peaks and tries to maintain the maximum of possible memory avaliable, and MGLRU follows an efficiency strategy and that the cost of swapping should be the lowest possible, allowing them to extract as much as available by making less costly decisions in the moment
  - Offer PPR as a backup reclaim on Snapdragon devices that have it, allowing PPR to better integrate with internal memory management by acting in just one situation: The main management is fragile and needs help! This is where PPR will come in, helping the system as a backup to prevent it from spending additional resources to avoid a crash, minimizing any impact. Useful for users who don't trust their internal memory management and want extra security
- Instead of panicking in extremely low memory situations, have the kernel prefer to use oom_killer and kill the most memory-consuming process. Doing this will significantly reduce OOM situations, allowing devices that suffer from constant OOM panics to suffer less from them, resulting in improved stability
- Integrate complete Android Runtime optimizations to optimize as much as possible the way Android handles compilation itself such as zygote optimizations that compress, create a critical window, and share data with each other with use of image cache for better app launch, and ensuring maximum precision for usage profiles, reducing unnecessary I/O and CPU by ensuring optimal time, including choosing optimal GC combinations based on compatibility, such as UFFD + CMC for current devices and CMC (background) and CC (foreground) for not-so-modern devices. It was not integrated into the compilation routine cleaning, this was left to Basic Cleaner, which is one of the recommendations I left at the beginning of the repository
- A comprehensive set of I/O improvements and optimizations for UFS and eMMC storage devices. These improvements improve I/O bursting & latency for UFS, I/O request grouping for EMMC, cache locality for hardware and freedom to hardware abstractions based on the needs of each block, reducing power consumption as well as improving latency and throughput, significantly reducing I/O stalls. In addition, it extracts the maximum of CFQ/BFQ schedulers in UFS/EMMC storage that use this, allowing you to reduce the weaknesses of these schedulers, allowing them to be more compatible with Flash storage (SSD) and not follow rigid disk strategies (HDD)
- The module provides customizable parameters, among them variable virtual memory sizes such as 0g up to 6g ZRAM, 0g to 3g Swapfile and 0g to 8g Write-back, also customizable compression & I/O algorithm selection, also includes the ability to configure the compression ratio aggressiveness of algorithms like zstd, deflate and lz4hc. Many parameters can be modified in the module panel by yourself, and will take effect after reboot. Pay attention to the comments
- Built-in inteligent memory extension: perform a small write-back when a used memory threshold is reached, and perform a large write-back only on the lock screen after reaching the number of small write-backs, based on the timer to check memory usage and without power consumption. Or you can choose to perform a large write-back after reaching the specified number of small write-backs. When the remaining RAM is less than the set threshold, a large write-back will be automatically performed after the screen locks to avoid phone lag
- Includes miscellaneous optimizations. Such as larger margin of error for lowmem devices, slightly reduces panic situations that may occur when a subsystem needs immediate synchronization but the system does not have space for it. Fixes for certain subsystems that were causing issues and regressions. Follow a Pinner Service proposal that pins only what is critical or essential to the system, leaving aside libs that the system can leave for later, with this Pinner Service tactic, jank is minimized by avoiding swapping libs that are critical, improving even the fluidity and opening of apps, incredible as it may seem, in addition to improving the sustainability of multitasking for a longer time. In addition to including adjshield, a way for you to protect your favorite apps (or whatever you want) from the evil lmk that kills them, working for all forms of lmk
- The module does not get along badly with third-party, on the other hand: It is recommended that you use third-party modules that can help in our way of memory management, of course, if they do not interfere directly with our optimizations, they are always welcome
- SELinux can still be enabled

## Requirement

- ARM64 (64-bit)
- Magisk, KSU or Apatch Most updated version if possible
- Android 8-15 (I don't know if my module is compatible with Android 16, but I would like users who use this version to report whether it is compatible or not)
- Have 2GB or more of memory in order to maximize the module's efficiency
- Recommended to use in SOCs with 8 cores, either big.LITTLE or DynamlQ
- May not be compatible with heavily modified/optimized ROMs and kernels. Please be aware of this if you are in these conditions
- It is recommended that the user have Busybox installed by default. This allows ZRAM to be manipulated with greater precision
- Disable any "RAM Plus" or options that affect ZRAM/Swap on your device when installing the module. Only leave them enabled if you see that they don't impact SkyScene or generate errors. If you feel like it, use them in conjunction with SkyScene if you want an additional improvement in memory management (if it improves, of course, in some ROMs, certain managers are more harmful than beneficial for multitasking)

## Installation

- Download the module from the Releases tab, not the zip from the repository. Then install this module, you will be greeted by the ability to choose the lmk you want, follow the instructions and follow the one you would like to use and reboot, there will also be additional optimizations that you are compatible with that will be inserted into your device, don't worry about them and then reboot. 
- Recently, users and developers have reported that several modules corrupted their systems after uninstallation. Like SkyScene, the reason for this is that current root managers are changing their code to something more integrated with the kernel to escape Google Play. The problem is that this has ended up generating horrible bugs, with some modules causing MASSIVE syntax errors when installing/uninstalling! So, if you installed SkyScene and would like to uninstall it, I apologize if your device suffers a bootloop; the problem will no longer be my problem, as it is already beyond what my module could cause.
- After reboot, open `/sdcard/Android/panel_memcfg.txt` to modify the module parameters, which will take effect after reboot.
- Open `/sdcard/Android/panel_adjshield.txt` to add the package name of the application that needs to be kept in the background, which will take effect after reboot.
- Some panel options only appear if compatibility is confirmed. Avoid using options that are not compatible with modern devices, older devices, other devices, or other privacy/security reasons.
- The default ZRAM size and other ZRAM & Swap optimizations are based on AOSP, which means these are the "default" values ​​for each amount of memory: 
  - 2GB of RAM gets 1GB of ZRAM by default. If the user has MGLRU, the ZRAM size is 1.2GB.
  - 3GB of RAM gets 1.5GB of ZRAM by default. If the user has MGLRU, the ZRAM size is 2GB.
  - 4GB of RAM gets 2GB of ZRAM by default. If the user has MGLRU, the ZRAM size is 2.5GB.
  - 6GB of RAM gets 3GB of ZRAM by default.
  - 8GB of RAM gets 4GB of ZRAM by default.
  - 12GB or more of RAM gets 6GB of ZRAM by default.
  - Default ZRAM algorithm is lz4. All algorithms are supported as long as the user has them in his Kernel. I highly recommend switching to zstd instead of sticking with lz4 for users with high-perf devices and powerful processors. This is because modern high-perf processors benefit MUCH more from storing as much data as possible in ZRAM, as they have enough processing power to do so. Keep the lz4 algorithm only if you need to save maximum CPU or notice lag spikes in games or the UI, but remember that you will notice a noticeable drop in multitasking.
  - ZRAM Dedup is enabled by default on systems with MGLRU. ZRAM Dedup is disabled by default on LRU systems, with the user being able to choose to keep it disabled or enabled.
  - Onboard memory expansion and Swapfile are disabled by default.
- ZSWAP is not currently supported.
- Simple LMK is not currently supported.
- Abandon CAF solutions for Snapdragon devices and focus on AOSP solutions. Avoid using things that Snapdragon considers best for its processors and allow memory management to be more integrated with the system, going one step further.
- The module includes not only modern AOSP optimizations and backward compatibility strategies, but also modern memory management strategies in ROMs from manufacturers such as Xiaomi and others.
- Lowmem devices are cell phones or tablets that have 4GB or less of RAM. High Perf Devices are cell phones or tablets that have 6GB or more of RAM.
- The cell phone that is being tested for optimizations is a lowmem (4gb with 2.5gb of ZRAM, lz4 and MGLRU), so I would appreciate it if high perf users reported their tests with the module in issues to see if their background apps are not being negatively impacted due to the "no stall tolerance" rule that I set for high perf devices.

## FAQ

### Sources

- [ZRAM Tuning and some other things](https://juejin.cn/post/7147284908367413261).
- [Cached and Phantom Process](https://github.com/agnostic-apollo/Android-Docs/blob/master/en/docs/apps/processes/phantom-cached-and-empty-processes.md)..
- [Studies on lmkd from the official Google page](https://source.android.com/docs/core/perf/lmkd?hl=pt-br).
- Some studies, especially on the I/O blocks and also the UFS/EMMC storage difference with HDD storage. In addition to studying ways to adapt CFQ and BFQ schedulers to flash storage.
- Some descriptions of what Pixel device users have experienced.
- Some reports and flags from AOSP master. Core optimizations are almost always based on AOSP itself, prioritizing modernization and efficiency above all else.
- Studies on human psychology using some channels of curiosities, school facts and some studies on human reflexes and perception.
- Tests and more tests with different configurations and in the same tests (multitasking, games, etc.), I spent hours of my days on this.
- And yes, the names of the SkyScene versions are based on the characters from the movie RIO. A movie set in Brazil, my homeland.

### Optimization recommendations based on manufacturer solutions (such as OnePlus, Samsung, Google, etc.)

- If you have little RAM and have LRU but want better multitasking, instead of using the AOSP amount, use the amount used by MGLRU devices:
- 2GB RAM = 1.2GB ZRAM
- 3GB RAN = 2GB ZRAM
- 4GB RAM = 2.5GB ZRAM
- These values ​​allow a slightly larger margin for these memory amounts, allowing up to 15% more data to be stored in ZRAM compared to the AOSP value. This is achieved using the lz4 compression algorithm, which is used by OnePlus due to its high compression and decompression speeds.
- For those who want to store as much data as possible in memory, use Aosp or Oneplus ZRAM values, but use the zstd compression algorithm. With zstd, you trade off higher CPU usage, slower compression and decompression speeds and possible stalls at high memory usage (around 80-85% ZRAM usage), with the amount of compressed data double or even triple compared to lz4, however, don't follow this idea if you have a weak processor, have at least one that is minimally competent, because zstd can be a bit problematic on these weaker processors.
- Use inteligent writeback as used on Samsung devices, but use writeback values ​​as low as 2GB to 6GB. Above these values, you may have useless "extra space."
- The same tips (less ZRAM size) for lowmem devices apply to high-perf devices. The difference with inteligent writeback is that a larger spacing between writes/frees is recommended to capture as much idle data as possible and store it. Recommended between 120 seconds (the system's basic reclaim time, synchronizing the two) or every 10 minutes. The longer the time, the less impact on power consumption and the greater the capacity to group idle data to send to storage, which can free up significant ZRAM depending on the situation.
- If you're a "Google-like" user, set the writeback to be applied every hour, which is Google's recommended standard.
- If you have UFS storage and the I/O algorithm "mq-deadline", use this algprithm, it is one of the MAYBE best I/O algorithms for stock devices with UFS Storage (i.e. without extremely optimized kernel).

#### General

- Due to the way SkyScene was previously built, your device may experience cache overflow more quickly when installing or updating SkyScene repeatedly. This has been partially fixed in more recent versions, but these annoying situations still occur. To completely fix this, the only and main recommendation is to format the device. This way, SkyScene can extract the maximum memory management from your device without cache penalties.

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

Q. When should I use the swapfile or memory expansion? And can I use both?   
A. Use it only in one situation: you have a low-RAM device and want better multitasking. Conversely, NEVER use the swapfile and memory expansion simultaneously; use only one. And as a recommendation, use memory expansion because it is much less damaging to storage when using idle, unmoving data.

Q. Why should I use ZRAM? Many recommend disabling all swaps.  
A. Ignore them; disabling ZRAM would lead to much worse performance, even affecting the storage's lifespan. Be aware that ZRAM can improve storage I/O performance by having less data written to the I/O, because ZRAM can already satisfy the memory demand that traditional memory could not. And yes, storage, whether with or without a swapfile, is used to save data, and this is called "Writeback." So much so that desktop users, even with 12GB or more of RAM, report dramatic improvements in responsiveness when using a measly 512MB of ZRAM. So, as a recommendation: if you don't want to use ZRAM, use a measly 512MB just to keep your storage healthy.

Q. Why do modern high-performance devices use the zstd algorithm instead of lz4? Why is lz4 recommended for low-mem devices and older high-performance devices?
A. Simple. In the past, most processors were too weak to handle all system tasks, so using algorithms like zstd, which consumed an absurd amount of CPU at the time, wasn't worth it. However, with modern processors today, zstd has become an increasingly viable and plausible option. Samsung's SDM 700 and 800 series processors started using this algorithm instead of lz4, allowing for a dramatic increase in multitasking on these more modern devices. So, a basic recommendation: if you have a powerful processor and 6GB or more of RAM, use the zstd algorithm; your multitasking will be IMMENSELY grateful.

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

Q. Kernel devs reported that MGLRU is too aggressive. Why?    
A. Ignore them. These devs ported an extremely inefficient MGLRU. Many of them even used thrashing prevent, which in an Android environment is extremely inefficient and even degrades MGLRU performance because Android doesn't handle waits well. It is dynamic and needs to be agile and fast to handle memory management.

Q. Why does SkyScene's Pinner service prioritize what's essential to the system?   
A. Simple. My Pinner service prioritizes pinning to memory what's truly important to the system and can't be delayed, like certain system_server libs and others. The goal is to minimize stalling of the main system thread by giving it the essential libs it uses for everything from the start.

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
@lululoid -- I am very grateful for the customize.sh functions that allowed me the ability to change lmkd during installation, because of that I will recommend its module in my repository as a way to help you and thank you indirectly  
@Iamlooper -- For the magisk MMT Reborn template. Thanks to the template, I was able to replace the old qti-mem-opt template and keep all the features without extra additions! Also now the cpu usage of the module has reduced by 2%, little but useful  

