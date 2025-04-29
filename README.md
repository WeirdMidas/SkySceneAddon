# SkyScene Addon

Memory management optimization for current Android platforms. Optimizing four central aspects: cached processes, battery consumption, scalable swapping and keeping as many applications as possible in the background, of course, respecting the system limits to avoid unnecessary OOM and Thrashing, so my dear user, RESPECT YOUR CELL PHONE!

SkyScene is based on the idea of ​​this module here: https://developer.aliyun.com/article/1230689.

The goal of SkyScene is to resemble this Chinese module, but adapted for devices that do not use Scene 8 because it is paid, and also for devices with different amounts of memory (2gb or more). This makes SkyScene focus mainly on energy savings and system smoothness over memory management. This means that SkyScene focuses on being adaptable to all memory demands, light, moderate, heavy multitasking and even games, always prioritizing battery savings and consistency. But always respecting the limits of the user's hardware, never going beyond to avoid inconveniences such as OOM, Thrashing and even crashes or PANIC because an app was killed by lmkd by mistake and unbridled push from the user.

## Feature

- Pure memory management optimization module, not containing other placebo and supporting all mainstream platforms
- Solve the problem that the background can't hang even if the free memory is large, by modifying device_config specified ActivityManager CUR_MAX_EMPTY_PROCESSES
- Customizable list of protected APPs, preventing them from being killed by Android in-userspace lowmemorykiller
- Fixed system common files in the file page cache, which significantly reduced the stucks caused by the key cache being swapped out due to page cache fluctuations
- The module achieves the effect of backstage anti-killing by expanding intelligent memory rewriting and modifying the memory management mechanism (lmkd & psi). By combining minfree's thresholds and replacing vmpressure with psi metrics, it makes minfree able to react to abnormal pressure more accurately. And by improving the backend process management, the effect of making the system run more smoothly and saving energy is achieved
- Prohibits kernel memory recycling threads for 7 cores, avoiding using all cores and reducing priority so they prefer small cores, reducing power consumption by up to 15mW
- Swapping dynamically scales based on usage and is user-centric, enabling the system to recover memory even under heavy pressure. This allows users to run more apps simultaneously while improving the overall efficiency of memory compression. We aim to maintain an optimal compression ratio of 2.8x or higher, depending on the processor’s capabilities
- Introduce hybrid swap! A virtual memory management technique that combines swapfile + zram + ppr, allowing swapping costs to be reduced by up to 27% and still increasing effective memory by 17% even with 1gb of swapfile, It also exponentially reduces writes to storage by allowing a more beneficial swapfile reclaim. As such, it can only be used to its full potential on devices with PPR (per-process reclaim) which are only found on phones with qualcomm processors. And hybrid swap is also required to activate the swapfile (if you use it together with ZRAM), if you do not have a snapdragon processor, hybrid swap can still be used, but only swapfile + zram, excluding ppr from the list (excluding swapping costs, reduced storage degradation and improved effective memory increase)
- Customizable ZRAM size and compression algorithm(needs kernel support), ranging from 0G to 6G
- Customizable swapfile size (needs hybrid swap active), ranges from 0G to 3G
- SELinux can still be enabled

## Requirement

- ARM64-v8a (64-bit)
- Magisk or KSU
- Android 12-15
- Have 2GB or more of memory in order to maximize the module's efficiency

## Installation

- Do not download the module via zip from the repository! Until I know when to send all the files in a single commit, it will be out of date
- Magisk or KSU
- Install this module and reboot, open `/sdcard/Android/panel_memcfg.txt` to modify the default parameters like ZRAM size, compression algorithm and even enable hybrid swap and swapfile size, and it will take effect after reboot
- Open `/sdcard/Android/panel_adjshield.txt` and add the package name of the APP that needs to be kept in the background. It will take effect after reboot.
- The default ZRAM size is as follows:
  - 2GB of ram gets 1GB of ZRAM by default
  - 3GB of ram gets 1.5GB of ZRAM by default
  - 4GB of ram gets 2GB of ZRAM by default
  - 6GB of ram gets 3GB of ZRAM by default
  - 8GB of ram gets 4GB of ZRAM by default
  - 12GB or more of ram gets 6GB of ZRAM by default
  - Hybrid swap and swapfile in general are set to 0 (i.e. disabled) by default, and the user needs to manually enable them via the panel
- ZSWAP is not currently supported
- ZRAM Writeback is not supported at the moment, and there is a chance that it will not be supported because my device cannot use ZRAM writeback even though it is compatible and has even been compiled into the kernel (if anyone knows how to do this, please help me)
- The lmk used for optimization is lmkd minfree and psi, where both are combined. If you have old lmk, or even simple lmk, they will not be optimized, showing no support for them

## FAQ

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
@lululoid -- extra lmkd adjustments, if you want RAM management totally focused on PSI, it is recommended to use its module (LMKD-PSI-ACTIVATOR) which is more recommended for devices with high-end cell phones that do not want so much "customization".
