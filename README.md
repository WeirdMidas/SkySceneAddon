# SkyScene Addon
![1000003588](https://github.com/user-attachments/assets/e02b145a-bba9-41a2-baea-5a9946051137)

Memory management optimization for current Android platforms. Optimizing four central aspects: cached processes, battery consumption, resource usage and keeping as many applications as possible in the background, of course, respecting the system limits to avoid unnecessary OOM and Thrashing, so my dear user, RESPECT YOUR CELL PHONE!

SkyScene is based on the idea of ​​this module here: https://developer.aliyun.com/article/1230689.

The goal of SkyScene is to resemble this Chinese module, but adapted for devices that do not use Scene 8 because it is paid, and also for devices with different amounts of memory (2gb or more). This makes SkyScene mimic the power saving and smoothness aspect of the Chinese module mentioned above, but adds an additional layer on top. This additional layer combines aspects such as power saving, smoothness and adaptability to most, if not all, memory demands where the management respects the user, and especially their hardware limits, never going beyond what it is capable of handling, to avoid problems such as OOM, Thrashing and even "Panics" such as some user app being killed because of pushing your limits and not respecting them and generating failures in sequence. In general, SkyScene values ​​a more efficient and adaptable memory management for the user and the hardware of their device in question. Allowing the system to decide when and where to kill apps or compress data with more efficiency and precision. Of course, always respecting limits such as processor, memory, I/O and other limits.

Furthermore, SkyScene combines these aspects, as well as aspects such as: user-friendly customization, which means that the user can customize certain module options to allow him to more roughly fit the management to his needs and hardware limits in question. Such as choosing the lmkd management mode when installing the module, customizing ZRAM parameters, using swapfile and other customizable mechanisms that the user wants to change. Even if it is a "light" customization at first, it still allows the user to customize some parameters to his needs.

## ⭐ Features

- Pure memory management optimization module, not containing other placebo and supporting all mainstream platforms
- Now, when installing the module, the user can choose between lmkd psi, lmkd minfree or lmkd psi + minfree when installing the module. It is not possible to choose another after selecting an lmkd, unless you reinstall the module again.
  - **lmkd psi**: Said by Google as the future "main" solution to replace minfree.
  - Advantages: Psi on supported phones is much more accurate in knowing which apps to kill, when using metrics that look at CPU, MEM and I/O. Based on these metrics, lmkd can choose who to kill to prevent stalls from occurring, making psi almost 2x-10x with fewer false positives.
  - Weaknesses: It can be much slower to make decisions against OOM or thrashing, and on devices with less memory, if they are not recent/have ROMs that optimize lmkd and its metrics, psi can detect any slight pressure as a potential "stall", killing user apps unnecessarily.
  - **lmkd minfree**: Similar to the old lmk, lmkd's minfree uses not just minfree, but minfree + customizable adj. This means that an app has more priority to be killed if it falls into the predefined group.
  - Advantages: It is much more stable and less prone to OOM and thrashing situations, due to it having predefined limits and being able to react with much less latency to any memory pressure. Allowing the user to avoid suffering from OOM or thrashing much less frequently than with psi.
  - Disadvantages: unlike psi, minfree has up to 10x more chances of having false positives for any pressure. Making it terrible without being optimized to maintain apps.
  - **lmkd psi + minfree**: Combines psi metrics with minfree levels and priorities. This allows lmkd to react to any pressure without depending on vmpressure, making minfree smarter to avoid reacting to any pressure.
  - Advantages: Combines minfree stability with psi perception, allowing the user the ability to react to almost any pressure because psi already knows the system limits and their priorities due to minfree. It is the combination of minfree's OOM and thrashing security + psi perception.
  - Disadvantages: Even though it is very advantageous, it takes the weaknesses of both lmkds, the difference is that they "mix" and reduce their weaknesses slightly.
  - In terms of the advantage between lmkds, here it is in terms of number of open apps:
lmkd psi > lmkd minfree + psi > lmkd minfree
  - In terms of security against OOM, Thrashing and freeing up memory to be able to play:
lmkd minfree > lmkd minfree + psi > lmkd psi
  - In terms of compatibility and likelihood of working more efficiently with devices (android 12+): lmkd minfree > lmkd minfree + psi > lmkd psi
  - In terms of playing games (in terms of freeing up processes that consume CPU to give everything to the game): lmkd psi > lmkd minfree + psi > lmkd minfree
  - **Warning**: The metrics for each lmkd management mod shown are more accurate on newer devices. As the device gets weaker and older, psi can become worse than even minfree and its combined version of the two when it comes to keeping apps open for longer (like in my case).
- Each lmkd, in addition to one of them being able to be selected as the main one when installing the module, is optimized through an "anti-kill" mechanism, which means that the lmkd used is optimized to avoid killing apps unnecessarily, prioritizing demand and respect for the user. It may not be perfect because some devices may have different needs, or the compatibility of some lmkds (mainly psi) may be lower than others, it's all a matter of how well each lmkd is implemented in your ROM. Devices with more memory can take full advantage of these adjustments, devices with less memory are less noticeable in these improvements. That's why the module offers the ability for the user to choose the lmkd he wants to use, in order to allow greater capacity for the user to maintain more compatibility with his device, in order to maximize the efficiency of the module
- Solve the problem that the background can't hang even if the free memory is large, by modifying device_config specified ActivityManager CUR_MAX_EMPTY_PROCESSES. With predefined limits for each amount of memory respecting the hardware limits:
  - For 2GB, 32 cached/background apps are set instead of the standard 24 for that device.
  - For 3-4GB, 64 cached/background apps are set instead of the standard 32 for that device.
  - For 6GB, 96 cached/background apps are set instead of the standard 32 for that device.
  - For 8GB or more, 128 cached/background apps are set instead of the standard 32 for that device.
- Customizable list of protected APPs, preventing them from being killed by Android in-userspace lowmemorykiller
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations
- It prohibits kernel memory recycling threads from running on all cores, preventing kswapd and oom_reaper from using all cores, where kswapd uses only four small cores with increased priority to maximize swap response, and oom_reaper on seven cores for stability purposes, keeping the priority of the two above normal threads but slightly below UX threads (118-116), reducing power consumption and allowing better response to pressure without the two preempting each other unnecessarily
- Optimizations and improvements to ZRAM that make it a bit better and be used to compress data that is actually useful at the moment, with improvements such as avoiding compressing hard-to-compress pages and preventing possible stalls, ideal size for each amount of memory and use of a default compression algorithm that many benchmarks consider as "ideal", as well as improvements in other subsystems of my module (such as lmkd, cached apps) to allow less useless/garbage data to be sent to ZRAM, in addition to kswapd working more consistently and avoiding preemption. Make the lowest compression ratio of ZRAM reach 2.8x, and try to reach default compression levels above 3.1x
- Additional compression in background apps. It is not ZRAM, it is a technique called CompAction. This memory management technique compresses apps that the user is using, if necessary: ​​the compressed app is sent to ZRAM, the app data is double compressed. Each amount of memory has the period and minimum amount of memory that each app uses to compress via CompAction:
  - 2GB of RAM the app needs to use 32mb to compress. With the compression period occurring every 30 seconds.
  - 3GB of RAM the app needs to use 64mb to compress. With the compression period occurring every 45 seconds.
  - 4GB of RAM the app needs to use 96mb to compress. With the compression period occurring every 45 seconds.
  - 6GB of RAM the app needs to use 128mb to compress. With the compression period occurring every 60 seconds.
  - 8GB of RAM the app needs to use 192MB to compress. With the compression period occurring every 90 seconds.
  - 12GB of RAM or more the app needs to use 256MB to compress. With the compression period occurring every 90 seconds.
- Introduce hybrid swap! A virtual memory management technique that combines swapfile + zram + ppr, allowing swapping costs to be reduced by up to 27% and still increasing effective memory by 17% even with 1gb of swapfile, It also exponentially reduces writes to storage by allowing a more beneficial swapfile reclaim. As such, it can only be used to its full potential on devices with PPR (per-process reclaim) which are only found on phones with qualcomm processors. And hybrid swap is also required to activate the swapfile (if you use it together with ZRAM), if you do not have a snapdragon processor, hybrid swap can still be used, but only swapfile + zram, excluding ppr from the list (excluding swapping costs, reduced storage degradation and improved effective memory increase)
- Customizable ZRAM size and compression algorithm(needs kernel support), ranging from 0G to 6G
- Customizable swapfile size (needs hybrid swap active), ranges from 0G to 3G
- SELinux can still be enabled

## Requirement

- ARM64-v8a (64-bit)
- Magisk or KSU Most updated version if possible
- Android 12-15
- Have 2GB or more of memory in order to maximize the module's efficiency

## Installation

- Do not download the module via zip from the repository! Until I know when to send all the files in a single commit, it will be out of date
- The project is being done more as "personal development", it is not made to be completely public, I only made it public so that when I release my first release I can have some opinion/feedback. Therefore, updates may take time. I am doing everything little by little, I have things to do and personal problems to solve in my personal life, so don't fight me to update things
- When installing the module in Magisk Manager or KSU, you will be able to choose between these three lmkd: lmkd psi (volume up), lmkd minfree (volume down) and lmkd minfree + psi (power button). To choose, press the physical buttons as I mentioned above
- Install this module and reboot, open `/sdcard/Android/panel_memcfg.txt` to modify the default parameters like ZRAM size, compression algorithm and even enable hybrid swap and swapfile size, and it will take effect after reboot
- Open `/sdcard/Android/panel_adjshield.txt` and add the package name of the APP that needs to be kept in the background. It will take effect after reboot
- The default ZRAM size is as follows:
  - 2GB of RAM gets 1GB of ZRAM by default
  - 3GB of RAM gets 1.5GB of ZRAM by default
  - 4GB of RAM gets 2GB of ZRAM by default
  - 6GB of RAM gets 3GB of ZRAM by default
  - 8GB of RAM gets 4GB of ZRAM by default
  - 12GB or more of RAM gets 6GB of ZRAM by default
  - Hybrid swap and swapfile in general are set to 0 (i.e. disabled) by default, and the user needs to manually enable them via the panel
- ZSWAP is not currently supported
- ZRAM Writeback is not supported at the moment, and there is a chance that it will not be supported because my device cannot use ZRAM writeback even though it is compatible and has even been compiled into the kernel (if anyone knows how to do this, please help me)
- The lmk used for optimization is lmkd minfree and psi, where both are combined. If you have old lmk, or even simple lmk, they will not be optimized, showing no support for them
- How to see if you are compatible with lmkd psi, write this command in termux with su mode:
zcat /proc/config.gz | grep PSI. If you find CONFIG_PSI, you are compatible, if you find together: CONFIG_PSI_FTRACE=y, your lmkd psi has an even higher precision than before.

## FAQ

### Recommendations

If you want a more "ingenious" memory management module than mine, check out lululoid's module

- https://github.com/lululoid/LMKD-PSI-Activator

Certainly if you like lmkd psi, using it instead of mine should be more beneficial, after all it specializes in that.

### How is the module tested for each version?

Simple, initially my device couldn't handle 6 apps for a long time. So in the Yuki-Chan version (the first one I'll release) I'll try to keep 6 apps active in memory for 30 minutes. And with each version of the module, I'll try to add +2 apps to last 30 minutes. And so on, I'll always try to keep my multitasking stable.

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
@unintellectual-hypothesis -- by hybrid swap and some ZRAM diskzise  
@lululoid -- I am very grateful for the customize.sh functions that allowed me the ability to change lmkd during installation, because of that I will recommend its module in my repository as a way to help you and thank you indirectly  
@Iamlooper -- For the magisk MMT Reborn template. Thanks to the template, I was able to replace the old qti-mem-opt template and keep all the features without extra additions! Also now the cpu usage of the module has reduced by 2%, little but useful
