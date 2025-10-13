---
title: Hanvon N10 mini (2022) 探索汇报
date: 2025-10-12T18:36:37.066Z
---


国庆爆肝通关 P5R 一周目，要上课了才从书包里发现我当时鬼迷心窍买了的写字板。考虑到目前应该有资源探索其上的软硬件设施，便花了数天与夜晚捣鼓了个七七八八，才得以在此时将这些结果化为文字。

## 设备简介

Hanvon N10 mini (2022) 是基于 Rockchip RK3566 平台的一款墨水屏产品。官方材料称，正面屏幕规格为 7.8 英寸，300dpi，支持 16 级灰阶算法，可模拟 256 级灰阶效果，支持 4096 触控笔。正面屏幕不支持基于常见的电容触摸，而全依赖于厂商的数字化仪电磁笔设施实现触控。我手上这款设备是 2+32G 配置，平平无奇。

设备的最新版本软件基于 Android 11 (R)，且安全补丁停留在 2021-06。

## 入门

### 安装第三方应用

使用蓝牙传输文件即可，但该方法过于低效了。建议使用 [CatShare](https://github.com/kmod-midori/CatShare)，该应用大致兼容国内的互传联盟制定的协议 —— 结合了蓝牙近场通信与 Android 自带的 WiFi Direct 实现。

> [!tip]
> 该应用同样支持在启用 [Shizuku](https://github.com/RikkaApps/Shizuku) 并相应授权后发送文件。

### 启用 ADB

厂商有意隐藏 AOSP 自带的设置应用，通过 [Pico-4/Settings](https://github.com/Pico-4/Settings) 即可简单的开启；

> [!note]
> 据该应用的 README 称其主要被开发以启动由字节跳动开发的 Pico 4 VR 设备上的 AOSP 设置应用，我推测同样的原理对 N10 设备也适用，事实确实如此。

在*关于平板电脑*中点击版本号 7 次启用开发者选项和 ADB 调试，并使用正确的 USB 线缆即可通过 ADB 操作该设备。  
在此基础上，使用 [tangoapp](https://app.tangoapp.dev/) 的本地应用版本能获得更优秀的调试体验。该在线应用的免费功能足够我们使用了。

### 获取更新包

通过 [Reqable](https://reqable.com) 和 [VPN Hotspot](https://github.com/Mygod/VPNHotspot)，可将经由本机 WLAN 热点连接网络的设备的请求截获下来。触发 N10 设备上的厂商 OTA 更新检测，便能够得到厂商更新包的 URL 来源。

> [!note]
> 该地址被硬编码于设备上包名为 `hanvon.aebr.hvsettings`，名称为 `设备设置` 的系统应用。
> ```bash
> $ rg UPDATE_INFO_URL
> hvsettings/sources/hanvon/aebr/hvsettings/system/DownloadView.java
> 120:    private static String UPDATE_INFO_URL = "http://edu.hwebook.cn/soft/xys/update.xml";
> 121:    private static String UPDATE_SYS_URL = "http://edu.hwebook.cn/soft/xys/";
> 122:    private static String UPDATE_SYS_BASE_URL = "http://edu.hwebook.cn/soft/xys/";
> 152:        UPDATE_INFO_URL = "http://note.api.hwebook.cn/sysupdate/update.xml";
> 154:            UPDATE_INFO_URL = "http://eink.hv.rcplay.cn/sysupdate/update.xml";
> 156:        UPDATE_SYS_BASE_URL = "http://cdn.zyeink.hwebook.cn/soft/xys/";
> 704:                                    String unused2 = DownloadView.UPDATE_SYS_BASE_URL = "http://cdn.zyeink.hwebook.cn/soft/xys/";
> ```

无论以何种办法，得到的 `http://note.api.hwebook.cn/sysupdate/update.xml` 即为所需。更新页面提示本机的 HWV 为 `15.00`，返回 XML 内容中仅有一例相关项。将对应 XML 项的 `domain` 和 `filename` 内容拼接得到一 zip 文件地址，下载后提取其 `boot.img` 文件即可。

### 安装 Magisk

保存该 `boot.img` 原件，在设备上安装 [Magisk Alpha](t.me/magiskalpha) 并根据其提示安装 Magisk，本机即得到 Root 权限。  
在此建议保留该 `boot.img` 原件，甚至更极端的情况下，保留获得到的更新包本件。

### 救砖

当因为奇怪的原因导致了 bootloop，通过 Rockchip 平台提供的 Download Mode 即可重置或备份系统。长按电源键整机关机后，按住设备上返回圆键的情况下按住电源键开机，与设备连接的开发端应显示有名为 USB download gadget 的设备出现。

作者使用的 MacOS 不需要安装对应的驱动（基于 `libusb` 通信），但 Windows 平台需要根据瑞芯微提供的资料安装驱动，在此恕不提供相关资料，作为题目供读者自由研究。

与 Rockchip 平台的 USB download gadget 通信的工具至少流传着三种，但实际测试时唯有第三方的 [DorianRudolph/rkdeveloptool](https://github.com/DorianRudolph/rkdeveloptool) 运作正常，且其命令行界面较其他发行更规范的多。

通常情况下，你只需要写 `boot` 分区 —— 通常也是我们更改了的分区即可。

```bash
$ build/rkdeveloptool write-partition boot magisk_patched-30200_TBIw6.img
$ build/rkdeveloptool reboot
```

该工具同样支持 `read` 与 `read-partition` 指令，但我尚没有对我手头上的这台设备测试过，尚认为是整机备份的方案。

## 进阶

我希望将其用作我 macbook 的输入设备使用，利用其高性能且特殊化的数字化仪硬件设施，所以我首先探索了该设备上具备有该方面能力的 Android 应用。

在这独一个项目上我几乎用完了我整个 2025.10 的 GitHub Copilot 高级请求配额，难顶。

### 相关系统应用

根据简单的阅览应用列表，发现有数个应用符合条件，这里选取其中看上去体量最小的*全局备注*（`com.hanvon.screennote`，包含 *note* 关键字，体积 944KB）。  
通过 `jadx -d` 反编译，注意到其包含有原生库 `libhw_PenEngine.so` 与 `libhw_PenDraw.so`。使用 [LibChecker](https://github.com/LibChecker/LibChecker) 筛查后发现其他有相关功能的系统应用也具备这两个库。观察发现有引用到这两个库的 Java 类。

```bash
@screennote $ rg loadLibrary
sources/hanvon/aebr/penengine/HWPenDraw.java
11:            System.loadLibrary("hw_PenDraw");

sources/hanvon/aebr/penengine/HWPenEngine.java
33:            System.loadLibrary("hw_PenEngine");
```

`HWPenDraw` 类中仅存在与电磁屏功能不相关的 `drawPath32` 与 `drawPath32Color` 方法，而 `HWPenEngine` 类中则有两个方法 `initialize` 和 `destroyEngine` 是由 JNI 调用的。  
通过 IDA 打开看上去最有相关性的 `libhw_PenEngine.so`，但其中大部分的是与绘制、笔刷等绘制功能，几乎找不到与电磁屏数据的明确相关信息。

### 查阅系统服务

此时注意到系统层面也一样依赖于电磁笔触控，联想到 `framework.jar` 与 `services.jar` 中应该存在相关的内容。对其也通过 `jadx -d` 反编译，通过搜索 `Pen`、`Hv` 等关键字，得到 `android.os.IHvPenDrawService` 和 `android.os.IHvPenDrawListener`。发现 `IHvPenDrawService` 中的相关方法 `enablePen` 和 `IHvPenDrawListener.onPenTouchUpStatus`，后者显然可用于监听电磁笔抬起事件。

```java
void onPenTouchUpStatus(boolean z, float[] fArr) throws RemoteException;
```

尝试编写一个应用，通过 Binder 连接到 `IHvPenDrawService`，并实现 `IHvPenDrawListener.onPenTouchUpStatus` 以接收数据，发现这些方法全是 hidden-api，用户态的应用无法用正常的办法连接；虽然我们还有 [RikkaApps/Shizuku](https://github.com/RikkaApps/Shizuku) 和捏造 AIDL 或者全部类声明抄一遍之类的办法来强行连接并获取数据，但仅能获取到 `onPenTouchUpStatus` 事件与采样点的数组/切片 —— 当且仅当笔尖离开屏幕后才能受到事件。这个办法还引发了一个意想不到的副作用：当调用 `enablePen` 后，屏幕上开始绘制出笔迹。  
做出结论：这样的实现完全不适合用于实现输入设备的能力。事已至此，似乎已经没法利用系统本身的用户态能力，我们需要更加深入了。

通过对 `services.jar` 的反编译，我们最终定位到对 `IHvPenDrawService` 的实际实现 `com.android.server.hvpendraw.HvPenDrawService`，以及 JNI 实现 `com.android.server.hvpendraw.NoteJNI`。  
尽管我们已经足够深入，我们还是发现 `NoteJNI` 提供了例如 `native_set_pen` 和 `native_set_pen_color` 等能力，且其调用的原生库名为 `paintworker`，而没能提供直接读取电磁屏数据的能力。

`libpaintworker` 位于 `/system/lib64/libpaintworker.so`，我们将其 `adb pull` 下来并使用 IDA 打开……根据这个名称，我们似乎不能从这里得到什么明晰的答案，无论是查阅其包含的符号，还是查阅其动态链接引用的其他库：

```bash
$ objdump -x libpaintworker.so | grep NEEDED
  NEEDED          libjpeg.so
  NEEDED          libpng.so
  NEEDED          libcutils.so
  NEEDED          libandroidfw.so
  NEEDED          libutils.so
  NEEDED          libbinder.so
  NEEDED          libui.so
  NEEDED          libskia.so
  NEEDED          libgui.so
  NEEDED          liblog.so
  NEEDED          libjnigraphics.so
  NEEDED          libc++.so
  NEEDED          libc.so
  NEEDED          libm.so
  NEEDED          libdl.so
```

不过我们起码是有线索的。`libpaintworker.so` 包含了这几个关键函数：`close_device(char const*)`、`init_getevent(void)`、`uninit_getevent(void)`、`get_event(input_event *，int)`、`open_device(char const*)`。  
简单阅读 `init_getevent` 的反编译结果：它遍历 `/dev/input/event*`，并查找设备名称里含有 `pen` 的设备。找到并打开设备，我们可以使用 `get_event` 读出其提供的数据。

### 深入输入设备

我们可以用 zig 实现类似的逻辑，显然 `/dev/input/event*` 是字符设备，可以用 `ioctl` 得知其设备名称。zig 提供了简单实用的交叉编译到 `aarch64-linux-android` 这样一个 triplet 的能力，且可不依赖于平台 libc。通过这一实现，我们成功的从 `/dev/input/event7` 读得数据。
通过模仿原始实现里的逻辑，我们也能构造出简单的解码逻辑……然后就出了问题。

- `pressure` 大概率会归零，猜想是由于读取速度过快的原因，或许实际实现上需要以更新而非每次都重新解码的方法组织；
- `tool=eraser` 的情况也类似，先传递了 `tool` 再传递了 `action=down`。这要求我们用状态机等办法去维护。

解码逻辑围绕着 `action=down` 和 `action=up` 两个关键帧。观察得到的数据，我注意到 `tool=eraser` 后立刻传递了 `action=down`。

```
event type=0x01 code=0x0141 value=   +1 => x=    0.000 y=    0.000 pressure=   0.000 action=undefined tool=eraser
event type=0x01 code=0x014a value=   +1 => x=    0.000 y=    0.000 pressure=   0.000 action=     down tool=   pen
```

以此可以写出这样的伪代码：

```
Tool = enum { pen, eraser }
Action = enum { up, down, undefined }

latestTool: Tool = .pen
latestPressure: float = .0
latestAction: Action = .undefined

fn onEvent(pressure: float, action: Action, tool: Tool) {
    latestTool = tool
    
    if (pressure != 0.) {
	    latestPressure = pressure
    }
    
    if (action == .down) {
        latestAction = .down
	    emit(touch=true, tool=latestTool, pressure=latestPressure)
    } else if (action == .undefined) {
	    if (latestAction == .down) {
	        emit(touch=true, tool=latestTool, pressure=latestPressure)
	    }
    } else {
        latestTool = .pen
        latestPressure = .0
        latestAction = .up
        
        emit(touch=false, tool=.no, pressure=.0)
    } 
}
```

如上所述，解码的问题并不是不能解决，事实上我也在这里提出了解决办法。但之后有更麻烦的问题需要解决：我发现与我的三星平板不同的是，这台 N10 mini 在笔尖并未接触到屏幕本身时即存在压力值并使得 `touch=true`，实测时也发现这种情况下确实直接将这一情况视作一次 touch，显然不符合直觉。  
更奇怪的是，存放压力值的位数仅有 10 位，这意味着其最大值 1023 不符合 Hanvon N10 mini 说宣传的具备 4096 级压感，而更贴近其官网上提到的更低端的 HW0808 电磁屏芯片（HW0868 最高只有 2048 所以也不太像）。我无从得知在这款设备内部到底是由什么芯片驱动，但这显然造成了极大的困惑。  
当时出现了这些问题的同时，我当时在想是否存在更深入的办法，便跟踪了是哪里的驱动维护了 `/dev/input/event7`。我顺便扫了一下输入设备的特性。

```bash
n10mini $ cat /proc/bus/input/devices
I: Bus=0013 Vendor=0002 Product=0000 Version=0100
N: Name="pen_touch"
P: Phys=input/hanvon/input0
S: Sysfs=/devices/virtual/input/input7
U: Uniq=
H: Handlers=event7 cpufreq
B: PROP=2
B: EV=b
B: KEY=40423 0 60000000 8000 0 0
B: ABS=1000003
```

按键事件与绝对坐标事件，没什么新奇的，考虑到三星设备也是用按钮实现；不过这里没有 `ABS_DISTANCE`，也就是距离屏幕距离，从硬件上或许就不是很能支持 Hover 用例了。  
在 `/vendor/lib/modules` 下有我们想要的东西：`hanvon_emr_int.ko`。EMR 即驱动电磁笔的数字化仪，通过 `lsmod` 也得以验证该模块得到加载，而 `rmmod` 会导致内核恐慌并自动重启。  
那么接下来的事情便了然了。

通过逆向 `hanvon_emr_int.ko`，发现其通过内核暴露了的（已验证）符号 `g_uart2_ops` 触发收到事件帧的回调，符号的类型是 `void (*)(unsigned char)` 的指针，接受一个 `u8`。这个串口一次传递一个字节的数据，而该内核模块中将每 7 个字节视作一个事件帧。  
也就是说只要能替换掉内核里这个函数指针，指向我们自己的函数就行 —— 在此基础上，只要我们的函数再去调用本来 `hanvon_emr_int.ko` 绑在这上的 `hw_interrupt` 函数就行。
但这也就是说，我们得开发一个内核模块了。

### 内核模块开发

为了开发一个内核模块，需要以下准备：

- 一个区分大小写的文件系统；
- 内核源码，与现有设备的版本相同或兼容；
- Module.symvers 文件。

MacOS 使用的 APFS 默认是大小写不敏感的，这极大程度的妨碍我们，因此需要使用 `hdiutil` 建立一个区分大小写的稀疏磁盘映像并挂载。

```bash
$ hdiutil create -size 6g -type SPARSEBUNDLE -fs 'Case-sensitive APFS' -volname linux-src linux-src.sparsebundle
$ hdiutil attach linux-src.sparsebundle
$ cd /Volumes/linux-src
```

挂载好后，我们可以切换到 `/Volumes` 下的对应挂载点处继续。

对于 N10 Mini，我们可以使用瑞芯微的开源内核而不是主线内核，后者会导致 `module_init` 失效等错误。

```bash
$ git clone https://github.com/rockchip-linux/kernel/ rockchip-kernel -b develop-4.19 --depth=1
```

我们需要使用 Linux 环境编译内核，对于 MacOS，这里推荐使用 [OrbStack](orbstack.dev) 提供的 Linux Machine 功能创建 Ubuntu 24.04 版本的容器。使用 `orb` 指令切换到容器中，并安装所需的软件包，在此我大致整理了一下我用到的 `apt install` 指令附上。

> [!tip]
> 中国大陆地区需要先更新镜像源并执行 `sudo apt update`。

```
ubuntu-24.04 $ sudo apt install gcc-aarch64-linux-gnu build-essential libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm bc
```

我们需要从设备上获取已有的 `.config` 并执行 `olddefconfig` 和 `modules_prepare`。

```
ubuntu-24.04 $ cd rockchip-kernel
ubuntu-24.04 @rockchip-kernel $ mac adb shell zcat /proc/config.gz > .config
ubuntu-24.04 @rockchip-kernel $ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- olddefconfig
ubuntu-24.04 @rockchip-kernel $ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules_prepare
```

内核模块的开发需要一个 `Modules.symvers` 文件，通过 [bol-van/extract-symvers-ng](https://github.com/bol-van/extract-symvers-ng)，该文件可从已有的 kernel 中导出。

首先，我们需要使用 `magiskboot` 解包我们现有的内核获得 Kernel Image，无论是原厂还是 Magisk-patched 都可以。根据 bol-van/extract-symvers-ng 的说明文档，我们需要首先获取内核的基址地址。

```bash
rk3566_eink:/ # sysctl kernel.kptr_restrict=1
kernel.kptr_restrict = 1
rk3566_eink:/ # grep text /proc/kallsyms | head
ffffff8008080000 T _text
ffffff8008080800 T _stext
ffffff8008080800 T __exception_text_start
ffffff800808139c T __exception_text_end
ffffff80080813a0 T __irqentry_text_end
ffffff80080813a0 T __irqentry_text_start
ffffff80080813a0 T __softirqentry_text_start
ffffff80080816e0 T __entry_text_start
ffffff80080816e0 T __softirqentry_text_end
ffffff8008083d0c T __entry_text_end
rk3566_eink:/ #
```

N10 Mini 暂且不需要修改 `cmdline` 添加 `nokaslr`，即使我验证过内核确实支持（`CONFIG_RANDOMIZE_BASE=y`），原因不明。总之我们得知内核基址是 `ffffff8008080000` 了。

根据 Magisk [文档](https://topjohnwu.github.io/Magisk/tools.html)，将 Magisk 安装包中的 `lib/arm64-v8a/libmagiskboot.so` 改名为 `magiskboot` 即可在 OrbStack 的 Linux Machine 中运行。  
使用 `magiskboot unpack <bootimg_filepath>` 解包 `boot.img`，在当前目录下得到 `kernel` 文件。

```bash
$ ls
dtb kernel ramdisk.cpio second
```

bol-van/extract-symvers-ng 提醒我们检查一下 `CONFIG_HAVE_ARCH_PREL32_RELOCATIONS`，我帮你们检查过了是  `=y`。根据其文档，我们执行指令：

```bash
$ python3 extract-symvers.py -b 64 -B ffffff8008080000 -k 4.19 -p y moded_kernel > Module.symvers
```

把得到的文件扔进内核源码根目录，准备工作终于搞定了。

> [!tip]
> 由于 `g_uart2_ops` 是从内核中导出的符号，直接 `__symbol_get` 和 `__symbol_put` 即可。

简单写一个 `/dev/hanvon_emr_raw` 字符设备驱动，我们就可以继续了。这部分之后会再附上。

### 用户态开发

现在直接从内核传来数据，其格式与 `/dev/input/event*` 有所不同，最大的不同是 tool 直接由头一个字节 `flag = frame[0]` 传递，省去了对 `latest*` 等历史状态的依赖，同时也便于达成另一个目的：实现符合直觉的点触检测。

> [!note]
> 我是先实现的这个再去问群友得到实现卡尔曼滤波这个办法的，但现在这个简单。

```zig
// 触摸判定状态机
const TouchDetector = struct {
    // 滞后阈值配置
    const TOUCH_THRESHOLD_RISING: u16 = 50; // 从 hover 到 touch 的阈值
    const TOUCH_THRESHOLD_FALLING: u16 = 30; // 从 touch 到 hover 的阈值
    const NOISE_FLOOR: u16 = 755; // 噪声底限（原始 hover 最大值）

    // 滑动平均窗口
    const WINDOW_SIZE: usize = 5;

    current_state: bool = false, // false = hover, true = touch
    pressure_history: [WINDOW_SIZE]u16 = [_]u16{0} ** WINDOW_SIZE,
    history_index: usize = 0,
    history_filled: bool = false,

    pub fn init() TouchDetector {
        return .{};
    }

    pub fn update(self: *TouchDetector, raw_pressure: u16) bool {
        // 添加新的压力值到历史记录
        self.pressure_history[self.history_index] = raw_pressure;
        self.history_index = (self.history_index + 1) % WINDOW_SIZE;
        if (self.history_index == 0) {
            self.history_filled = true;
        }

        // 计算滑动平均（平滑噪声）
        const smoothed = self.getSmoothedPressure();

        // 减去噪声底限，得到真实的触摸压力
        // 如果平滑后的值小于噪声底限，认为是 hover
        const adjusted_pressure: i32 = @as(i32, smoothed) - NOISE_FLOOR;

        // 滞后阈值状态机
        if (self.current_state) {
            // 当前是 touch 状态，需要降到 FALLING 阈值以下才变回 hover
            if (adjusted_pressure < TOUCH_THRESHOLD_FALLING) {
                self.current_state = false;
            }
        } else {
            // 当前是 hover 状态，需要升到 RISING 阈值以上才变成 touch
            if (adjusted_pressure > TOUCH_THRESHOLD_RISING) {
                self.current_state = true;
            }
        }

        return self.current_state;
    }

    fn getSmoothedPressure(self: *const TouchDetector) u16 {
        var sum: u32 = 0;
        const count: usize = if (self.history_filled) WINDOW_SIZE else self.history_index;

        if (count == 0) return 0;

        var i: usize = 0;
        while (i < count) : (i += 1) {
            sum += self.pressure_history[i];
        }

        return @intCast(sum / @as(u32, @intCast(count)));
    }

    pub fn getAdjustedPressure(self: *const TouchDetector) i32 {
        const smoothed = self.getSmoothedPressure();
        return @as(i32, smoothed) - NOISE_FLOOR;
    }
};
```

> [!note]
> 755 是把笔头拿掉再放在我 macbook 的托盘处磁铁上测出来的，事实上确实有一定的鲁棒性。
> ……这个词翻译的还是太烂了，每次都得吐槽一遍。

对 tool 的判定：

```zig
const is_eraser = (self.flag & 0x04) != 0;
```

parse 代码，附带位解析：

```zig
pub fn parse(bytes: [9]u8) Frame {
        const flag = bytes[0];

        // 根据 hw_interrupt 反编译结果和汇编代码:
        // frame_buf[0] = flag
        // frame_buf[1-2] = X 的高位和中位
        // frame_buf[3-4] = Y 的高位和中位
        // frame_buf[5] = pressure 低 7 位
        // frame_buf[6] = 混合字节: [7:6]=unused, [5]=X低2位的高位, [4]=X低2位的低位, [3]=Y低2位的高位, [2]=Y低2位的低位, [2:0]=pressure高3位

        // X 坐标 = (byte[1] & 0x7F) << 9 | (byte[2] & 0x7F) << 2 | (byte[6] >> 5) & 3
        const x = (@as(u16, bytes[1] & 0x7F) << 9) |
            (@as(u16, bytes[2] & 0x7F) << 2) |
            ((@as(u16, bytes[6]) >> 5) & 3);

        // Y 坐标 = (byte[3] & 0x7F) << 9 | (byte[4] & 0x7F) << 2 | (byte[6] >> 3) & 3
        const y = (@as(u16, bytes[3] & 0x7F) << 9) |
            (@as(u16, bytes[4] & 0x7F) << 2) |
            ((@as(u16, bytes[6]) >> 3) & 3);

        // 压力值 = (byte[6] & 7) << 7 | (byte[5] & 0x7F)
        const pressure = ((@as(u16, bytes[6]) & 7) << 7) |
            (@as(u16, bytes[5] & 0x7F));

        // const tilt_x = bytes[7] & 0x7F;
        // const tilt_y = bytes[8] & 0x7F;

        const action = Action.fromFlag(flag);
        const tool = Tool.fromFlag(flag);

        return Frame{
            .flag = flag,
            .x = x,
            .y = y,
            .pressure = pressure,
            // .tilt_x = tilt_x,
            // .tilt_y = tilt_y,
            .tool = tool,
            .action = action,
        };
    }
```

搞定，现在应该比较符合直觉了，如果感觉写的有压力就阈值改大概 `670` 就行。

### Tilt 值的读取

其实如果是用了 HW0868 的话，好像是有对笔身倾斜角度的上报的，`hanvon_emr_int.ko` 中有个结构叫 `hw0868_global`，包含了对 `tilt_x` 和 `tilt_y` 的上报。但 N10 Mini 一直上报为 0，只能当作不支持了。  
不过这也给了一个启示：说不定 `hanvon_emr_int.ko` 其实是多种设备通用的，只要是 Linux，哪怕是更狭隘的 Android 设备也都能支持。不过这就留待后续开发吧。

## 展望与疑问

传递的 pressure 显然只有 10 位，怎么做到 4096 级压感的呢，更像是用了 HW0808 芯片的样子……但也不是很符合。  
不过我没有也不打算拆机，手头也搞不到什么逻辑分析仪，就这样吧。  
之后我会考虑把现有的工作结果打成 Magisk 模块，那样部署就轻松多了，尽管现在本地的代码都未能整理完毕。在这之后我想我应该能实现一个简单的数位板之类的，不过在那之前还有很多的工作要做，比如说把现有 zig 实现重写到 rust 然后我们可以用 uniffi 生成 JNI Binding 不会那么痛苦，配套的 Android App 和 MacOS 应用等，此外可能还得用类似 DriveDroid 的思路，以减小软件接触面之类的。  
`libpaintworker` 和 `libhw_PenEngine` 似乎还具备了直接往墨水屏设备上绘制的能力（tangoapp 的 scrcpy 看不到正被绘制的笔迹），但我目前用不到这个能力，改天有点子了可以再来开发。  
无论如何，这些都是很久之后的事情了，后面代码整理完了我再在这里唠叨。

## 重要引用

本文提及的某些重要步骤严重依赖于这些文本提供的教程。

---

- Hommey, M. (2012, August 6). _Building a Linux Kernel Module without the Exact Kernel Headers_. Glandium.Org. https://glandium.org/blog/?p=2664
- Wu, J. (2024, November 8). _Magisk Tools_. Magisk Documentation. https://topjohnwu.github.io/Magisk/tools.html
