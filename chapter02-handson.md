♯ /proc/device-treeを確認

kenny@raspberrypi:~ $ cat /proc/device-tree/model
Raspberry Pi 5 Model B Rev 1.1k

kenny@raspberrypi:~ $ ls /proc/device-tree
'#address-cells'   chosen        hvs@107c580000     __overrides__     serial-number
 aliases           clk-108M      i2c0if             phy              '#size-cells'
 arm-pmu           clk-27M       i2c0mux            psci              soc@107c000000
 axi               clocks        interrupt-parent   pwr_button        __symbols__
 cam0_clk          compatible    leds               reserved-memory   system
 cam0_reg          cooling_fan   memory@0           rp1_firmware      thermal-zones
 cam1_clk          cpus          memreserve         rp1_vdd_3v3       timer
 cam1_reg          dummy         model              sd-io-1v8-reg     wl-on-reg
 cam_dummy_reg     firmwarekms   name               sd-vcc-reg

# カーネルのログ確認
dmesgのコマンドはカーネルが起動し栄光のHW初期化やドライバのメッセージを確認できる。

## カーネルメッセージをすべて表示
dmesg

## カーネルメッセージの末尾をリアルタイムで追う（新しいデバイス接続など）
dmesg --follow

## フィルタして特定のドライバやデバイスを見る
dmesg | grep usb
dmesg | grep camera(今回の実機ではなし)

## dmesgのリングバッファ
dmesgはカーネルが管理するバッファ利用域に出力されたログを表示する。
printk()と呼ばれる関数で、カーネル空間のコードがこのバッファ利用域に書き込みを行う。
この領域はリングバッファと呼ばれ、古いメッセージが新しいメッセージで書き換えられることがある。

dmesg | lessして、ログを俯瞰してみる

ログの長さは高々600L未満
```
kenny@raspberrypi:~ $ dmesg | cat | wc
    577    5104   43099
```

とりあえずGPTの言うキーワード周辺を見てみる。
- Booting Linux
> [    0.000000] Booting Linux on physical CPU 0x0000000000 [0x414fd0b1]
先頭にブート開始のログが出ている。

- Firmware: Raspberry Pi firmware
> [    0.000000] psci: PSCIv1.1 detected in firmware.
> [    0.020018] raspberrypi-firmware soc@107c000000:firmware: Attached to firmware from  2025-06-13T09:39:26, variant start_cd
> [    0.024021] raspberrypi-firmware soc@107c000000:firmware: Firmware hash is 5855b10b00000000000000000000000000000000
> [    1.195181] Bluetooth: hci0: BCM: firmware Patch file not found, tried:
> [    2.484788] rp1-firmware rp1_firmware: RP1 Firmware version eb39cfd516f8c90628aa9d91f52370aade5d0a55
> [    2.887954] brcmfmac: brcmf_c_preinit_dcmds: Firmware: BCM4345/6 wl0: Aug 29 2023 01:47:08 version 7.45.265 (28bca26 CY) FWID 01-b677b91b

種々のファームに関してログが出ているバージョンとかハッシュとかがダンプされている（ハッシュ確認して、どうするんだ？）

- bcm2712
今回のAPチップに関するログ
kenny@raspberrypi:~ $ dmesg | grep -i  bcm2712
>[    0.383144] bcm2712-iommu-cache 1000005b00.iommuc: bcm2712_iommu_cache_probe
[    0.523008] bcm2712-iommu 1000005100.iommu: bcm2712_iommu_init: DEBUG_INFO = 0x20804774
[    0.523368] bcm2712-iommu 1000005100.iommu: bcm2712_iommu_probe: Success
[    0.523759] bcm2712-iommu 1000005200.iommu: bcm2712_iommu_init: DEBUG_INFO = 0x20804774
[    0.524099] bcm2712-iommu 1000005200.iommu: bcm2712_iommu_probe: Success
[    0.524471] bcm2712-iommu 1000005280.iommu: bcm2712_iommu_init: DEBUG_INFO = 0x20804774
[    0.524812] bcm2712-iommu 1000005280.iommu: bcm2712_iommu_probe: Success
[    1.084491] vc4_hvs 107c580000.hvs: bcm2712_iommu_of_xlate: MMU 1000005200.iommu
[    1.084498] vc4_hvs 107c580000.hvs: bcm2712_iommu_probe_device: MMU 1000005200.iommu
[    1.084503] vc4_hvs 107c580000.hvs: bcm2712_iommu_device_group: MMU 1000005200.iommu
[    1.084511] vc4_hvs 107c580000.hvs: bcm2712_iommu_attach_dev: MMU 1000005200.iommu
[    1.084968] vc4-drm axi:gpu: bcm2712_iommu_of_xlate: MMU 1000005200.iommu
[    1.084973] vc4-drm axi:gpu: bcm2712_iommu_probe_device: MMU 1000005200.iommu
[    1.085578] vc4-drm axi:gpu: bcm2712_iommu_device_group: MMU 1000005200.iommu
[    1.085678] vc4-drm axi:gpu: bcm2712_iommu_attach_dev: MMU 1000005200.iommu
[    2.614382] pispbe 1000880000.pisp_be: bcm2712_iommu_of_xlate: MMU 1000005100.iommu
[    2.614389] pispbe 1000880000.pisp_be: bcm2712_iommu_probe_device: MMU 1000005100.iommu
[    2.614395] pispbe 1000880000.pisp_be: bcm2712_iommu_device_group: MMU 1000005100.iommu
[    2.614403] pispbe 1000880000.pisp_be: bcm2712_iommu_attach_dev: MMU 1000005100.iommu
[    2.667506] rpi-hevc-dec 1000800000.codec: bcm2712_iommu_of_xlate: MMU 1000005100.iommu
[    2.667514] rpi-hevc-dec 1000800000.codec: bcm2712_iommu_probe_device: MMU 1000005100.iommu
[    2.667520] rpi-hevc-dec 1000800000.codec: bcm2712_iommu_device_group: MMU 1000005100.iommu
[    2.667527] rpi-hevc-dec 1000800000.codec: bcm2712_iommu_attach_dev: MMU 1000005100.iommu

なにやってるんだろうか？

- vc-sm-cma
これら2つは引っかからなかった。

- v3d

```
kenny@raspberrypi:~ $ dmesg | grep -i v3d
[    0.835309] v3d 1002000000.v3d: [drm] Transparent Hugepage support is recommended for optimal performance on this platform!
[    0.841825] [drm] Initialized v3d 1.0.0 for 1002000000.v3d on minor 0
```

## Kernelとファームウェアの関係
Linux Kernel
 ├─ Driver（RP1）
 │    └─ RP1 firmware に接続し、バージョン取得・初期化ログを出す
 │
 ├─ Driver（Wi-Fi/brcmfmac）
 │    ├─ /lib/firmware/ からファームを読み込み
 │    ├─ デバイスに書き込み（アップロード）
 │    └─ バージョン情報やエラーを dmesg に出す
 │
 ├─ Driver（Bluetooth/hci_bcm）
 │    ├─ パッチファイルを探す
 │    ├─ ロード成功/失敗をログに記録
 │    └─ 動作準備ができたら HCI デバイスとして登録


ブートローダーは OS を起動できる状態までハードウェアを準備し、OS に制御を渡すための前座
Kernelはハードウェアを抽象化する。ハードウェアをOS内につないでアプリから利用可能にするための管理を行う。

【電源 ON】
     ↓
BootROM
     ↓
──── Bootloader（ハードの初期化、カーネルの準備） ────
     ↓
Linux Kernel（OSの心臓）
     ↓
systemd（ユーザ空間）
     ↓
アプリケーション