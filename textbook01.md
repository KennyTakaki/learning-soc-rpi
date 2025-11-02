# 🧩 頭の中で動かすラズパイ実験ノート（全9章）
> ― Raspberry Pi 5を題材に、ARM SoCとLinuxカーネルの基礎を「思考実験」で完全理解する ―  
> 著者：あなた（＋ChatGPT）

---

## 📘 概要

このノートは、Raspberry Pi 5を実際に持っていなくても、  
**ARM SoC／Linux カーネル／デバイスツリー／カメラインターフェース**の仕組みを体系的に理解するための仮想実験教材です。  

各章では「実際に触るような想定コマンド」→「何が見えるか」→「理解できる本質」をセットで学びます。

---

# 第1章　ハードウェアを見立てる：SoCという世界

### 🎯 目的
Raspberry Pi 5のボードを“頭の中で分解”し、SoCの構造を把握する。

### 🧠 イメージ
```
[BCM2712 SoC]
├─ Cortex-A76 CPU × 4
├─ GPU (VideoCore VII)
├─ ISP
├─ DDR Controller
├─ PCIe / USB / Ethernet
└─ GPIO / I²C / SPI / UART
```

### 🔍 学び
- **SoC = CPU + 周辺回路 + 通信バス**
- GPIOヘッダはSoCのピンを直接引き出したものである。
- ピンアサインを理解することは「電気信号の経路を読む」こと。

---

# 第2章　ブートを覗く：電源ONからLinux起動まで

### 🧠 シーケンス
```
BootROM → bootloader.bin → start.elf → kernel8.img → initramfs → systemd
```

### 💻 想定コマンド
```bash
dmesg | less
```

### 🔍 学び
- カーネルは`dmesg`にデバイス初期化ログを逐次出力。
- Linuxは「デバイスツリー」をもとにハードを認識している。
- 「OSがハードを理解する」仕組みを可視化できる。

---

# 第3章　GPIOを動かす：ファイルI/Oがハードを操る

### 💻 想定コマンド
```bash
echo 18 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio18/direction
echo 1 > /sys/class/gpio/gpio18/value
```

### 🧠 イメージ
`/sys`はユーザ空間からカーネルドライバへの仮想的な命令ポート。  

### 🔍 学び
- Unix哲学：「すべてはファイル」  
- ファイルI/Oがハードレジスタへの抽象インタフェース。  
- 他SoC（例：i.MX）でも同じ思想で動いている。

---

# 第4章　デバイスツリーを読む：ハードを記述する言語

### 💻 想定コマンド
```bash
dtc -I dtb -O dts -o bcm2712.dts /boot/firmware/bcm2712-rpi-5-b.dtb
```

### 💬 抜粋例
```dts
gpio@7e200000 {
    compatible = "brcm,bcm2712-gpio";
    #gpio-cells = <2>;
    gpio-controller;
    status = "okay";
};
```

### 🔍 学び
- Device Tree = **ハードウェアの構成表現**。  
- カーネルコードを変えずに構成を切り替えられる。  
- i.MXなど他SoCでも同じ構造（`.dtsi` 継承）。

---

# 第5章　カーネルをビルドする：BSPという概念

### 💻 想定コマンド
```bash
git clone --depth=1 https://github.com/raspberrypi/linux
cd linux
make bcm2712_defconfig
make -j$(nproc) Image modules dtbs
```

### 🔍 学び
- `arch/`, `drivers/`, `scripts/` など階層的構造。  
- `defconfig` はSoC固有設定＝**Board Support Package**。  
- カーネル構成を自分で再現できるようになる。

---

# 第6章　カメラを接続する：MIPI CSIの流れを追う

### 💻 想定コマンド
```bash
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --stream-mmap --stream-count=100 --stream-to=frame.raw
```

### 🧠 データフロー
```
Sensor(I²C) → MIPI CSI Receiver → ISP → V4L2 API → /dev/video0
```

### 🔍 学び
- MIPI CSIは物理層、Linuxはその上の**V4L2メディア層**を扱う。  
- ISPはセンサ出力を人間が見える画像に変換する信号処理器。  
- i.MX93ではISPを持たず、よりシンプルなISI構成になる。

---

# 第7章　カーネルモジュールを作る：ドライバの入口を知る

### 💻 想定コード
```c
#include <linux/module.h>
#include <linux/gpio.h>
#include <linux/delay.h>

static int __init toggle_init(void){
  int gpio=18;
  gpio_request(gpio,"LED");
  gpio_direction_output(gpio,0);
  for(int i=0;i<5;i++){gpio_set_value(gpio,1);msleep(300);
                       gpio_set_value(gpio,0);msleep(300);}
  return 0;
}
static void __exit toggle_exit(void){gpio_free(18);}
module_init(toggle_init); module_exit(toggle_exit);
MODULE_LICENSE("GPL");
```

```bash
make -C /lib/modules/$(uname -r)/build M=$(pwd) modules
sudo insmod toggle_gpio.ko
```

### 🔍 学び
- `module_init` / `module_exit` でカーネルに登録。  
- ドライバは「ハードをCで抽象化する構造体」。  
- Linuxの“デバイスモデル”概念を体感。

---

# 第8章　構造を再構築する：SoCとOSの地図

```
┌───────────────────────────┐
│ User Space                │
│ (bash, Python, libcamera) │
└────────────┬──────────────┘
             │ sysfs / ioctl
┌────────────┴──────────────┐
│ Kernel Space              │
│  ├ Device Drivers         │
│  ├ V4L2 / GPIO Subsystems │
│  └ Scheduler / MM / VFS   │
└────────────┬──────────────┘
             │ MMIO / IRQ
┌────────────┴──────────────┐
│ ARM SoC (BCM2712)          │
│  ├ CPU (A76×4)            │
│  ├ CSI / ISP / GPU         │
│  └ GPIO / I²C / SPI        │
└───────────────────────────┘
```

### 🔍 学び
- OSとハードの層構造を頭の中で再構築できる。  
- 「割り込み」「メモリマップI/O」「ドライバ登録」が一体化していると理解。  
- これがそのまま**i.MX シリーズのBSP構造**に対応する。

---

# 第9章　次のステップ：頭の中から実機へ

| 段階 | 内容 | 目的 |
|------|------|------|
| Step 1 | QEMUでARM64仮想環境を起動 | ブート体験を再現 |
| Step 2 | U-Bootをビルドしログを観察 | ブートローダ理解 |
| Step 3 | Yocto Projectで最小Linuxを構築 | 組込みBSP構築の練習 |
| Step 4 | i.MX93 EVKまたはJetsonで実験 | ISPや電源制御など実務層へ |

---

## 🧠 このノートで得られる力

- SoCを抽象化して理解できる「レイヤ思考」  
- Linuxカーネルの構造を俯瞰する読解力  
- ハードウェアをソフトウェアとして“再現”する想像力  

---

> 💬 **最後に**
>
> このノートの目的は「ラズパイを買うこと」ではなく、  
> **ラズパイを“頭の中で動かす”力を手に入れること。**  
>  
> それができれば、i.MX 9 シリーズやどんなSoCでも、  
> あなたの中で自由に動かせるようになります。

---
