# SkyScene Add-On
![1000007971](https://github.com/user-attachments/assets/07013c32-24b4-48db-9dbd-c36a49089755)

"Guys! Pose for the photo!"
## Modern and user-friendly memory management

Memory management optimization for current Android platforms. Optimizing five central aspects: 1. efficient reuse of cached processes to minimize reloads, 2. ease the reclaim and make it proactive instead of decisive, balancing pauses and significantly increasing throughput, 3. focus on reducing stalls coming from CPU, MEM and I/O and be consistent with the lmk modernization (LMKD) and if necessary, offer compatibility with old lmk for these changes and lastly: 4. favor multitasking, consider the limits of all devices and allow them to reach this limit, but of course: if the situation gets tight, allow the system to act to avoid serious problems such as OOM, Thrashing and even PANIC. This shows that the module respects the limitations of each device, and encourages the user to do the same, **RESPECT YOUR CELL PHONE!**

SkyScene is based on the idea of ​​this module here: https://developer.aliyun.com/article/1230689.

The goal of SkyScene is to resemble this Chinese module, but adapted for devices that do not use Scene 8 because it is paid, and also for devices with different amounts of memory (2gb or more). This makes SkyScene mimic the power saving and smoothness aspect of the Chinese module mentioned above, but adds an additional layer on top. This additional layer combines aspects such as power saving, smoothness and adaptability to most, if not all, memory demands where the management respects the user, and especially their hardware limits, never going beyond what it is capable of handling, to avoid problems such as OOM, Thrashing and even "Panics" such as some user app being killed because of pushing your limits and not respecting them and generating failures in sequence. In general, SkyScene values ​​a more efficient and adaptable memory management for the user and the hardware of their device in question. Allowing the system to decide when and where to kill apps or compress data with more efficiency and precision. Of course, always respecting limits such as processor, memory, I/O and other limits.

The module is made to have universal compatibility, optimizing the memory management of each device as much as possible, even if it is different, and even adding specific optimizations for the hardware and ROM that the device is located on. This means that the module focuses on only ONE thing: NO PLACEBO! In other words, the module tries to extract the MAXIMUM possible from the memory management that standard Android offers.

Furthermore, SkyScene combines these aspects, as well as aspects such as: user-friendly customization, which means that the user can customize certain module options to allow him to more roughly fit the management to his needs and hardware limits in question. Such as choosing the lmkd management mode when installing the module, customizing ZRAM parameters, using swapfile and other customizable mechanisms that the user wants to change. Even if it is a "light" customization at first, it still allows the user to customize some parameters to his needs.

## ⭐ Features

- Pure memory management optimization module, not containing other placebo and supporting all mainstream platforms. It also reduces energy consumption proportionally, allowing the user to enjoy better battery life
- Now when installing the module, you can choose to choose one of the three or four depending on your kernel versions of LMK in the module installation, where you will receive options to choose, at least for Android 10 or higher, where in Android 9 or lower it is fixed in the old LMK, the famous driver LMK. Each LMK below has its advantage in specific, and also dedicated optimizations for each one after the block below:
  - **lmkd psi**: Said by Google as the future "main" solution to replace minfree
  - Advantages: Psi on supported phones is much more accurate in knowing which apps to kill, when using metrics that look at CPU, MEM and I/O. Based on these metrics, lmkd can choose who to kill to prevent stalls from occurring, making psi almost 2x-10x with fewer false positives
  - Weaknesses: It can be much slower to make decisions against OOM or thrashing, and on devices with less memory, if they are not recent/have ROMs that optimize lmkd and its metrics, psi can detect any slight pressure as a potential "stall", killing user apps unnecessarily
  - **lmkd minfree**: Similar to the old lmk, lmkd's minfree uses not just minfree, but minfree + customizable adj. This means that an app has more priority to be killed if it falls into the predefined group
  - Advantages: It is much more stable and less prone to OOM and thrashing situations, due to it having predefined limits and being able to react with much less latency to any memory pressure. Allowing the user to avoid suffering from OOM or thrashing much less frequently than with psi
  - Disadvantages: unlike psi, minfree has up to 10x more chances of having false positives for any pressure. Making it terrible without being optimized to maintain apps
  - **lmkd psi + minfree**: Combines psi metrics with minfree levels and priorities. This allows lmkd to react to any pressure without depending on vmpressure, making minfree smarter to avoid reacting to any pressure
  - Advantages: Combines minfree stability with psi perception, allowing the user the ability to react to almost any pressure because psi already knows the system limits and their priorities due to minfree. It is the combination of minfree's OOM and thrashing security + psi perception
  - Disadvantages: Even though it is very advantageous, it takes the weaknesses of both lmkds, the difference is that they "mix" and reduce their weaknesses slightly
  - **old lmk**: it's the same as lmkd minfree. The difference is that it's much faster at killing apps and checking memory limits
  - Advantages: Much more stable and faster against OOM due to being a kernel driver. This generally allows old lmk to respond almost immediately to any memory pressure that exceeds its limits
  - Disadvantages: Same weaknesses as lmkd minfree, but faster overall and much less "technological", depending on the manufacturer that modified the lmk driver to make it useful
  - In terms of the advantage between lmks, here it is in terms of number of open apps:
lmkd psi > lmkd minfree + psi > lmkd minfree > old lmk
  - In terms of security against OOM, Thrashing and freeing up memory to be able to play: old lmk > lmkd minfree > lmkd minfree + psi > lmkd psi
  - In terms of compatibility and likelihood of working more efficiently with old devices: old lmk and lmkd minfree > lmkd minfree + psi > lmkd psi
  - In terms of playing games (in terms of freeing up processes that consume CPU to give everything to the game): lmkd psi > lmkd minfree + psi > old lmk > lmkd minfree
  - **Warning**: The metrics for each lmkd management mod shown are more accurate on newer devices. As the device gets weaker and older, psi can become worse than even minfree and its combined version of the two when it comes to keeping apps open for longer (like in my case)
- Each lmk (old lmk and three variants of lmkd), in addition to one of them being able to be selected as the main one when installing the module, is optimized through an "anti-kill" mechanism, which means that the lmk used is optimized to avoid killing apps unnecessarily, prioritizing demand and respect for the user, after all, the anti-killing optimizations are based on AOSP/Google, making the tweaks compatible with most devices. If you want to know what the optimizations are for each lmk management mode, see below:
  - **lmkd psi** (Android 10-12): Due to changes in the lmkd management strategy with the full implementation of the lmkd AOSP tweaks. PSI has been affected, now the killing method that PSI will exercise is: pushing the multitasking limits as much as possible, where the extreme usage and OOM/Thrashing limit has become even higher compared to its sister variants. Lmkd PSI can now push the maximum multitasking limit of the device, and react against stalls in real time. Now lmkd follows its original proposal, eliminating false positives as much as possible and keeping multitasking at its peak while using any free memory, after all "free memory is waste", as some internet users say
  - **lmkd psi + new strategy** (Android 13+): lmkd psi for new Androids adds an extra property that works only on them. With this property, a PSI management mode is activated, extremely faster, more responsive and slightly more accurate, using ALL memory pressure measurements equally and in cooperation with kernel reclaim to reclaim memory more efficiently. This reduces memory reclaim time by ~40-70%, reduces PSI's impact on the system and even eliminates the 10 second window of standard PSI. This makes PSI able to react to any pressure more accurately and proactively than before, allowing to push the limits of OOM and Thrashing in extreme memory usage a bit further. This form of lowmemorykiller made with PSI becomes the form of management that many might could call it extremely efficient and balanced, compared to kernel implementations that also seek to keep the system under control. The difference is that now PSI can push the limits a little further
  - **lmkd minfree** (Android 10-12): Now that minfree has become the epitome of stability, if you know the limits of your device, it is useful to use minfree to keep the chances of OOM or Thrashing at a minimum, however, the anti-killing applied in it is done to keep as much data as possible in memory, before triggering immediate killing. Useful for users who rely on the classic lmk implementation and still want to use it on their current devices, such as users with large amounts of memory, etc
  - **lmkd minfree + new strategy** (Android 13+): Now with the implementation of the new strategy, minfree has become much more accurate and faster in deciding what to kill, now prioritizing keeping processes that the user is currently using "intact", with minfree now being much faster in finishing and choosing which processes will be killed, allowing memory recovery that was previously fast to become much faster
  - **lmkd minfree + psi** (Android 10-12): Combines the stability behavior of minfree with the responsiveness of psi. You get a "perfect" form of memory management that recognizes the limits of your device, but allows you to react much more accurately to any pressures, where with the increased intelligence of PSI the system is able to react to stalls even more predictably when using the device's memory limits, already knowing its original limits lmkd becomes able to react fully respecting the device and not pushing it beyond what it originally can handle. It is perhaps the most balanced LMKD form for most, and all devices
  - **lmkd minfree + psi + new strategy** (Android 13+): Combining memory limits aspects with a much more efficient logic. Using all metrics equally, the lmkd combination has become a much more balanced memory management option, outperforming normal implementations thanks to combining the advantages and optimizations of minfree + psi with the new strategy. Making lmkd psi + minfree much more proactive and able to react to more memory usage situations than users would have been able to normally. Finally, this version of minfree + psi can be considered the CLOSEST to ideal behavior for Android devices, always keeping memory free but using it consciously and efficiently
  - **old lmk**: If the user has old lmk, it will still receive anti-kill improvements. However, unlike the others, it is focused on keeping as much data in RAM as possible, having optimized thresholds and triggers for memory reuse. Unlike the others, even keeping as much memory in use as possible, it is much faster and even efficient against OOM, where when it is close to occurring, old lmk will kill everything that is necessary, avoiding OOM situations faster. Old lmk will also make the limit between extreme use and OOM longer than lmkd minfree
- In addition, secondary optimizations that benefit lmk were added, such as pinning the reaper thread on big cores to speed up the cleaning of dead processes by up to +50%
- Solve the problem that the background can't hang even if the free memory is large, by modifying device_config specified ActivityManager CUR_MAX_EMPTY_PROCESSES. With predefined limits for each amount of memory respecting the hardware limits:
  - For 2GB, 32 cached/background apps are set instead of the standard 24 for that device
  - For 3-4GB, 64 cached/background apps are set instead of the standard 32 for that device
  - For 6GB, 96 cached/background apps are set instead of the standard 32 for that device
  - For 8GB or more, 128 cached/background apps are set instead of the standard 32 for that device
- Customizable list of protected APPs, preventing them from being killed by Android lowmemorykiller (both old lmk and lmkd)
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations
- If the device has UFS storage, optimizations are applied that improve memory flushing, reducing memory consumption for unnecessary dirty pages. If it is EMMC, it is the opposite, reducing unnecessary write operations to extend the storage life in exchange for less available memory
- Used zygote prefork to reduce possible CPU and I/O stalls when opening and switching apps in multitasking. Reducing the negative impact of these actions on the system
- Memory Reclaim's performance and throughput are much better compared to what Android's native system has, even on custom ROMs. Features like MGLRU that are likely to come in Android 12+ ROMs, SkyScene activates its full version because in these ROMs only a "basic" version is activated, which is significantly inferior to the full version, SkyScene also improves kswapd, making it more consistent and stable through parallelism that allows distributing kswapd tasks with less congestion with the added benefit of if the kernel supports it, scaling across idle cores for even greater parallelism throughput, its own cgroup to prevent kswapd from competing with background and even foreground tasks, priority below UX threads and core pinning that allows it to use small cores with more preference than big ones. All this overall reduces swapping costs by up to -53%, and with improvements in throughput of up to ~65% or more. If the user is not happy with the default kswapd optimizations, he can customize the various kswapd options via the module panel, allowing the user to adapt the reclaim to his own taste (more throughput, less latency, battery, slower or faster swapping, etc.)
- Integrated intelligent memory expansion writeback: Every time a specified percentage of used memory is reached, small writes are made until the memory demand is satisfied. If the demand is not satisfied after a while, a large write is made, as a form of debit to try to satisfy the demand completely and prevent the device from entering rebound due to large memory usage. Due to the way it is executed, power consumption only occurs in high memory usage, and due to the way it was compiled, battery consumption is low in checking and executing the writes, only occurring a higher consumption when large writes occur, but the user can customize the execution of the intelligent writeback to their liking completely, such as the percentage at which it should start executing small and large writes, time to start, limits and other more adverse settings
- This module does not conflict with third-party modules that perform secondary memory management optimizations, AS LONG AS they DO NOT INTERVENE with our optimizations and are secondary and additional optimizations. So much so that the module itself integrates third-party modules that are selected by the author based on quality and that can fully integrate with the module's objective, obviously thanking and giving credit to the original developers. If you meet the requirements of these third-party modules, it is recommended that you use them together with the main one to ensure more effective memory management, control of background processes, reduction of duplicate data and battery saving
- Introduces Per-Process Reclaim based on the PRLMK kernel driver. This means that the Hybrid Swap strategy has changed to something more suited to the current system needs and also user references. This means that PPR can now perform well even if the user uses only ZRAM, after all the PPR strategy has changed where it applies proactive reclaim based on "efficiency" that reaches a higher success rate level than Hybrid Swap, reaching +85% average reclaim efficiency. As such, it can only be used on Qualcomm devices and is mainly recommended for devices with 4GB or less, but users with more memory can use it too, if they want more efficiency in the memory usage of their processes
- Customizable ZRAM and Swapfile size, compression algorithm, whether or not to use ZRAM dedup also added ZRAM Writeback (needs kernel support) with the ZRAM size ranging from 0G to 6G, Swapfile size ranges from 0G to 3G and the size of the writeback going from 0G to 4G. And the module will try to have the lowest compression rate of 2.8x, with the attempt to achieve average compression equal to or above the ideal rate of modern Androids of 3.1x
- SELinux can still be enabled and also contain a panel at: storage/android/panel_memcfg.txt. Where the user can change customizable parameters and also know specific debugging information, such as whether your reclaim is modern or old, how modern your memory management is and other quirks

## Requirement

- ARM64-v8a (64-bit)
- Magisk or KSU Most updated version if possible
- Android 8-15
- Have 2GB or more of memory in order to maximize the module's efficiency

## Installation

- Do not download the module via zip from the repository! Until I know when to send all the files in a single commit, it will be out of date
- The project is being done more as "personal development", it is not made to be completely public, I only made it public so that when I release my first release I can have some opinion/feedback. Therefore, updates may take time. I am doing everything little by little, I have things to do and personal problems to solve in my personal life, so don't fight me to update things
- When installing the module in Magisk Manager or KSU, you will be able to choose between these three lmkd: lmkd psi (volume up), lmkd minfree (volume down) and lmkd minfree + psi (power button). To choose, press the physical buttons as I mentioned above
- In case you're wondering, some lmkd optimizations are "universal" that work equally for all devices, and there are also lowmem (for devices below or equal to 4gb of RAM) and high performance (for devices above or equal to 6gb)
- Install this module and reboot, open `/sdcard/Android/panel_memcfg.txt` to modify the default parameters like ZRAM size, compression algorithm and even use swapfile, and it will take effect after reboot
- Open `/sdcard/Android/panel_adjshield.txt` and add the package name of the APP that needs to be kept in the background. It will take effect after reboot
- The default ZRAM size and other ZRAM & Swap optimizations are based on AOSP, which means these are the "default" values ​​for each amount of memory:
  - 2GB of RAM gets 1GB of ZRAM by default
  - 3GB of RAM gets 1.5GB of ZRAM by default
  - 4GB of RAM gets 2GB of ZRAM by default
  - 6GB of RAM gets 3GB of ZRAM by default
  - 8GB of RAM gets 4GB of ZRAM by default
  - 12GB or more of RAM gets 6GB of ZRAM by default
  - Default ZRAM algorithm is lz4, user can choose other algorithms that are available on their devices after installation
  - ZRAM Dedup is disabled by default because it is completely dependent on the user's workload type
  - ZRAM Writeback comes in at 0 (i.e. disabled in both options) with the user choosing to enable it and choose whatever size they want
  - ZRAM Writeback comes with a writeback daemon that is fully configurable in its functions, with the parameters being available on the panel
  - ZRAM Writeback is only recommended for devices with UFS storage. Devices with EMMC storage are not recommended because random latency spikes may occur depending on the situation
  - Swapfile are set to 0 (i.e. disabled) by default, and the user needs to manually enable via the panel
- ZSWAP is not currently supported
- The lmk used for optimization in the module are: Old LMK and LMKD. Other LMK are not compatible such as Simple LMK and derivatives
- How to see if you are compatible with lmkd psi, write this command in termux with su mode: zcat /proc/config.gz | grep PSI. If you find CONFIG_PSI, you are compatible, if you find together: CONFIG_PSI_FTRACE=y, your lmkd psi has an even higher precision than before
- To find out if your kernel has compatibility with ZRAM Dedup, use this command in Termux with su: zcat /proc/config.gz | grep ZRAM, if CONFIG_ZRAM_DEDUP=y appears, it means that your kernel is compatible to use dedup, if this flag does NOT appear with "=y", it means that your kernel does NOT have it, or it is blocked by the kernel because the kernel developers did not want to go ahead with this optimization, leaving it unable to be activated without the user or the dev recompiling the kernel with the flag activated (with "=y")
- Specific optimizations based on compatibility. Like UFFD GC for kernels that have it but it is disabled and some that are to avoid adding and resulting in nothing, like MGLRU that can now recognize that the device has MGLRU to apply the flag for all generations
- Only use the second third-party Cirno if you have Freeze Cgroup V2 (usually found in snapdragon kernels 4.19 or higher and in normal kernels 5.4 or higher). And want a recommendation? If you have these requirements, use Cirno, it saved a lot of battery on my cell phone

## FAQ

### Sources

- [ZRAM Tuning and some other things](https://juejin.cn/post/7147284908367413261)
- [Cached and Phantom Process](https://github.com/agnostic-apollo/Android-Docs/blob/master/en/docs/apps/processes/phantom-cached-and-empty-processes.md)
- [Per-Process Reclaim based on LMK driver](https://github.com/darkhz/prlmk/tree/README)
- [Studies on lmkd from the official Google page](https://source.android.com/docs/core/perf/lmkd?hl=pt-br)
- LMKD optimization are based on AxionOS Custom Rom. Only that with customization and improvements in the optimization they apply by respecting the use of memory in various situations
- Some custom kernels that optimized kswapd and implemented the "kswapd threads" patch
- Tests and more tests with different configurations and in the same tests (multitasking, games, etc.), I spent hours of my days on this

### Recommendations

If you want a more "ingenious" memory management module than mine, check out lululoid's module

- https://github.com/lululoid/LMKD-PSI-Activator

Certainly if you like lmkd psi, using it instead of mine should be more beneficial, after all it specializes in that.

If you don't want to use my module or use lululoid's module but want extra management (and you have android 8-14), use OneB1nk's A1Memory, it's just special thanks I can give to him, recommending his project for everyone to see.

- https://github.com/OneB1ank/A1Memory?tab=readme-ov-file

If you have kernel 4.19 (on a Qualcomm/Snapdragon device) or kernel 5.4 and android 12+, you can choose to use [Cirno](https://github.com/Freezer-Team/Cirno), a "module-app" that freezes processes that the user is not using at the moment/in the recents tab automatically, this will reduce a little the energy consumption and proportionally the RAM consumption. It can be useful to fully control the background processes.

### Tips for imitating some Google-like aspects

If you want to slightly imitate Google's memory management (like Google Pixels), here are some recommendations, follow only those that you find coherent for your management style and performance and energy saving needs:

1. 2 or 4 threads for kswapd. Some Google devices focused on energy saving use 2 threads instead of 4.

2. Use small cores with preference for kswapd. Instead of using cores 0-6 (the module's default), use cores 0-3 or 0-5 (depending on whether your phone is a 4x4 or 6x2). This is because Google prefers to put swapping in the "background" instead of a "proactive" task in freeing up memory.

3. Use lmkd psi + minfree, Google currently uses lmkd with the combination of psi and minfree, even though psi is in theory a "total replacement for minfree". This is because minfree can end up saving the device from OOM situations where lmkd was too slow to save the device.

4. Use the zstd or lz4 algorithm depending on the number of kswapd threads you chose. 4 threads = zstd, 2 threads = lz4. This is because the stronger the algorithm used, the more threads are recommended to avoid congestion, the lighter the algorithm, the fewer threads can perform with less total energy consumption.

5. Keep half of the RAM as ZRAM + Use zram writeback instead of swapfile. Avoids latency spikes and ends up integrating better with ZRAM.

### Tips for using Cirno to its full potential

Here are some tips for using Cirno (Freeze Cgroup) to its full potential:

1. ONLY mark apps that you tend to use frequently in the whitelist. Cirno currently has a bug that does not unmark apps from the whitelist, so keep that in mind. Apps that you only use for one function and do not tend to use it together with your multitasking, do not need to be added.

2. Mark your games, preferably all or most of them.

3. To mark apps in the whitelist, after granting all permissions (lsposed, magisk, etc.), click on the app icon and press the toggle option, the translation of it is whitelist, which will put the app in the whitelist. The whitelist will start after reboot. However, there is a bug that makes it impossible to remove an app from the whitelist, but the developer is fixing this.

These tips will be here until the Cirno developer fixes the whitelist bugs. I have left them here only as a recommendation for users who want to use Cirno but may encounter problems.

### Random tips that might help

- If you have a 6x2 device (6 small cores and 2 big ones) and want maximum swapping performance while ignoring the energy cost. You can choose to set the affinity to 0-5 and 6 kswapd threads. With this, your kswapd will reach throughput levels that can exceed 60% just in parallelism, if combined with uclamp or schedtune's prefer idle, its own cgroup and other kswapd optimizations, you can reach almost +90% swapping throughput.

- If you have the zstd compression algorithm, only use it in two situations: if you want to hold as much data as possible in memory, for example to last longer in intensive multitasking. Or if you work a lot with processes that use a lot of memory continuously. With the use of the zstd algorithm, in exchange for a higher CPU usage, you get a compression rate almost twice as high as lz4, and if you also use ZRAM Writeback, the data sent to them is reduced, making the writeback file have much more space to be used.

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
@lululoid -- I am very grateful for the customize.sh functions that allowed me the ability to change lmkd during installation, because of that I will recommend its module in my repository as a way to help you and thank you indirectly  
@Iamlooper -- For the magisk MMT Reborn template. Thanks to the template, I was able to replace the old qti-mem-opt template and keep all the features without extra additions! Also now the cpu usage of the module has reduced by 2%, little but useful  
@OneB1ank -- Thanks to him for his third-party management module. I don't know how to thank him because I couldn't get in touch with him, so I hope he doesn't get mad at me and understands my use. I give him full credit to the point that I recommend his module in my repository   
@FreezeTeam Project -- Thanks to them for Cirno. The second additional third-party in my module, I really appreciate it and as a way of rewarding it follows the same dynamic as the others     
@yinwanxi -- Thanks for libcgroup script as a tool to optimize threads separately    
