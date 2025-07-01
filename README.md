# SkyScene Add-On
![1000007971](https://github.com/user-attachments/assets/07013c32-24b4-48db-9dbd-c36a49089755)

"Guys! Pose for the photo!"
## Modern and user-friendly memory management

Memory management optimization for current Android platforms. SkyScene is based on the idea of ​​this module here: https://developer.aliyun.com/article/1230689.

## ⭐ Features

- The module can avoid background interruption by modifying the memory management mechanism (lmk, psi), providing four optimized forms of these mechanisms that the user can choose when installing the module, as follows:
  - lmkd psi: Uses the psi form mentioned by Google, making it closer to a 'complete replacement for minfree' as Google tries to integrate it, lmkd psi comes with optimizations that allow it to become more accurate, even on devices with less memory, recognizing thresholds more accurately and respecting the user above all. PSI will work so that it will respect the user's multitasking as much as it can
  - lmkd minfree: Minfree will work like the classic minfree from old lmk, but with many more thresholds, and even watermark checking is included. As said, lmkd minfree will work in a way that it tries to keep the device out of OOM situations as much as possible, it is like a guard that does not let the client get into too much danger
  - lmkd minfree + psi: Combining the minfree + psi strategy, with this form of management, it can be said that lmkd comes closest to an "ideal" form of app killing, because it has the protection of minfree with the scalability of psi, allowing the user to have the benefits of both lmkds working simultaneously without leaning towards either side
  - old lmk: For devices with the old lmk via driver, such as devices with Android below 10. They will also receive optimizations that will be designed to keep as much data in memory as possible, before deciding to kill something. Also included is the ability for the user to choose to use the old lmk or lmkd if they are on Android 10+ and still have the old lmk driver
- It also includes the differentiation between "lowmem vs high performance" devices, where both vary in their optimizations, which are:
  - Lowmem: In lowmem devices, LMK becomes more tolerant and relies more on reclaim, where it tends to kill processes based on whether reclaim can save in moments of memory pressure. If it cannot, LMK kills based on how much the system is under pressure, having a critical limit where it kills more aggressively
  - High Perf: Less tolerant to stalls, if they occur, it tends to kill more aggressively. This difference may be weak with lowmem devices, but due to the larger amount of memory in high perf devices, this type of device ends up relying more on its ability to allocate memory than on "unpredictable things". This means that high perf devices tend to rely more on their amount of memory, allowing it to be used in its entirety
- And based on improving the management of background processes, we can't forget to optimize kswapd, which is optimized to align with the scheduler, allowing kswapd to respond to demand almost immediately while avoiding migration failures, cache misses, and other penalties. This overall reduces the probability of processes being killed by up to 70%, increasing multitasking dramatically
- It also includes optimizations on how many background processes the system can allow. Depending on the amount of memory on the device, the number of cached/background processes can scale, following this style:
  - 2GB can have up to 32 cached/background apps instead of 24
  - 3GB/4GB can have up to 64 cached/background apps instead of 32
  - 6GB can have up to 96 cached/background apps instead of 32-64
  - 8GB or more can have up to 128 cached/background apps instead of 32-64
- The module provides customizable parameters, among them variable virtual memory sizes such as 0g up to 6g ZRAM, 0g to 3g Swapfile and 0g to 8g Write-back, also includes optimization of VM parameters and customizable compression algorithm selection. Many parameters can be modified in the module panel by yourself, and will take effect after reboot. Pay attention to the comments
- Built-in inteligent memory extension: perform a small write-back when a used memory threshold is reached, and perform a large write-back only on the lock screen after reaching the number of small write-backs, without any delay or power consumption. Or you can choose to perform a large write-back after reaching the specified number of small write-backs. When the remaining RAM is less than the set threshold, a large write-back will be automatically performed after the screen locks to avoid phone lag
- Also integrated optimizations in PPR (Per-Process Reclaim) that mimics the behavior of prlmk, this overall allows PPR to be used more efficiently and is less likely to cause drawbacks in user processes, with the cache hit rate being above 85%
- Includes miscellaneous optimizations. Such as the inclusion of Zygote Prefork to reduce possible CPU and I/O stalls when the device opens or switches apps. Pinning of essential Android libs, to reduce Jank that occurs when system libs are swapped unintentionally. In addition to including adjshield, a way for you to protect your favorite apps (or whatever you want) from the evil lmk that kills them, working for all forms of lmk
- The module does not get along badly with third-party, on the other hand: It is recommended that you use third-party modules that can help in our way of memory management, of course, if they do not interfere directly with our optimizations, they are always welcome
- And finally, It also includes keeping SElinux active, of course, it was not tested on all devices, but on my device SElinux remained activated

## Requirement

- ARM64-v8a (64-bit)
- Magisk or KSU Most updated version if possible
- Android 8-15
- Have 2GB or more of memory in order to maximize the module's efficiency

## Installation

- Download the module from the Releases tab, not the zip from the repository. Then install this module, you will be greeted by the ability to choose the lmk you want, follow the instructions and follow the one you would like to use and reboot, there will also be additional optimizations that you are compatible with that will be inserted into your device, don't worry about them and then reboot. 
- After reboot, open `/sdcard/Android/panel_memcfg.txt` to modify the module parameters, which will take effect after reboot.
- Open `/sdcard/Android/panel_adjshield.txt` to add the package name of the application that needs to be kept in the background, which will take effect after reboot.
- The default ZRAM size and other ZRAM & Swap optimizations are based on AOSP, which means these are the "default" values ​​for each amount of memory: 
  - 2GB of RAM gets 1GB of ZRAM by default.
  - 3GB of RAM gets 1.5GB of ZRAM by default.
  - 4GB of RAM gets 2GB of ZRAM by default.
  - 6GB of RAM gets 3GB of ZRAM by default.
  - 8GB of RAM gets 4GB of ZRAM by default.
  - 12GB or more of RAM gets 6GB of ZRAM by default.
  - Default ZRAM algorithm is lz4.
  - Onboard memory expansion and Swapfile are disabled by default.
- ZSWAP is not currently supported.
- Simple LMK is not currently supported.

## FAQ

### Sources

- [ZRAM Tuning and some other things](https://juejin.cn/post/7147284908367413261).
- [Cached and Phantom Process](https://github.com/agnostic-apollo/Android-Docs/blob/master/en/docs/apps/processes/phantom-cached-and-empty-processes.md).
- [Per-Process Reclaim based on LMK driver](https://github.com/darkhz/prlmk/tree/README).
- [Studies on lmkd from the official Google page](https://source.android.com/docs/core/perf/lmkd?hl=pt-br).
- LMKD optimization are based on AxionOS Custom Rom. Only that with customization and improvements in the optimization they apply by respecting the use of memory in various situations.
- Some descriptions of what Pixel device users have experienced.
- Tests and more tests with different configurations and in the same tests (multitasking, games, etc.), I spent hours of my days on this.

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
