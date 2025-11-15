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

# `/boot`, `/lib/modules`, `/proc` の**役割**を整理
/boot
GPUファームがFAT32のパーティションにあるデータをもとにbootした結果、Linux 上では /boot として表現されている。

/lib/modules
カーネルは2種類のドライバを扱う。built-inかloadable modules（*.ko）　　
このうちloadable modulesを保存しているが/lib/modules/<kernel version>

```
kenny@raspberrypi:~ $ ls /lib/modules
6.12.25+rpt-rpi-2712  6.12.25+rpt-rpi-v8  6.12.47+rpt-rpi-2712  6.12.47+rpt-rpi-v8
```

４つのカーネルがあるが、実際には起動しているカーネルと一致しているものが利用されている。他は過去バージョンのものだったり。

```
kenny@raspberrypi:~ $ uname -r
6.12.47+rpt-rpi-2712
```

----
① カーネル or udev が .ko を自動ロードする
② insmod / modprobe が .ko をカーネル空間に読み込む
③ ドライバの init() / probe() が実行される
④ デバイスにマッチし、バインドされる
⑤ /proc/modules に登録される
⑥ dmesg に “probe” ログが出る

https://www.aps-web.jp/academy/linux-rtos/wr-linux/23638/

----
ユーザ空間（userland）

/usr/bin/modprobe   ← 実行ファイル（コマンド）
    ↓ 依存解析/alias解決
    ↓ カーネルへロード要求 (system call)

カーネル空間（kernel space）

-----------------------------------------

init_module() / finit_module()
    ↓
モジュールをメモリに配置
    ↓
シンボルのリンク
    ↓
モジュールの init() 実行
    ↓
デバイスにマッチすれば probe() 実行

----


カーネルに読み込まれているモジュール一覧

```
cat /proc/modulesの結果
```

<名前> <サイズ> <使用カウント> <依存モジュール> <状態> <メモリアドレス>

実際のファイル /lib/modules/$(uname -r)/... にあるrfcomm.ko, snd_seq.ko などに対応する。

<details>
kenny@raspberrypi:~ $ cat /proc/modules
rfcomm 81920 4 - Live 0x0000000000000000
snd_seq_dummy 49152 0 - Live 0x0000000000000000
snd_hrtimer 49152 1 - Live 0x0000000000000000
snd_seq 98304 7 snd_seq_dummy, Live 0x0000000000000000
snd_seq_device 49152 1 snd_seq, Live 0x0000000000000000
algif_hash 49152 1 - Live 0x0000000000000000
algif_skcipher 49152 1 - Live 0x0000000000000000
af_alg 49152 6 algif_hash,algif_skcipher, Live 0x0000000000000000
bnep 49152 2 - Live 0x0000000000000000
binfmt_misc 49152 1 - Live 0x0000000000000000
spidev 49152 0 - Live 0x0000000000000000
brcmfmac_wcc 49152 0 - Live 0x0000000000000000
aes_ce_blk 49152 4 - Live 0x0000000000000000
aes_ce_cipher 49152 1 aes_ce_blk, Live 0x0000000000000000
ghash_ce 49152 0 - Live 0x0000000000000000
gf128mul 49152 1 ghash_ce, Live 0x0000000000000000
sha2_ce 49152 0 - Live 0x0000000000000000
sha256_arm64 49152 1 sha2_ce, Live 0x0000000000000000
brcmfmac 376832 1 brcmfmac_wcc, Live 0x0000000000000000
brcmutil 49152 1 brcmfmac, Live 0x0000000000000000
sha1_ce 49152 0 - Live 0x0000000000000000
rpi_hevc_dec 65536 0 - Live 0x0000000000000000
cfg80211 1032192 1 brcmfmac, Live 0x0000000000000000
pisp_be 49152 0 - Live 0x0000000000000000
sha1_generic 49152 1 sha1_ce, Live 0x0000000000000000
v4l2_mem2mem 65536 1 rpi_hevc_dec, Live 0x0000000000000000
videobuf2_dma_contig 49152 2 rpi_hevc_dec,pisp_be, Live 0x0000000000000000
videobuf2_memops 49152 1 videobuf2_dma_contig, Live 0x0000000000000000
videobuf2_v4l2 49152 3 rpi_hevc_dec,pisp_be,v4l2_mem2mem, Live 0x0000000000000000
videodev 344064 4 rpi_hevc_dec,pisp_be,v4l2_mem2mem,videobuf2_v4l2, Live 0x0000000000000000
raspberrypi_hwmon 49152 0 - Live 0x0000000000000000
videobuf2_common 98304 6 rpi_hevc_dec,pisp_be,v4l2_mem2mem,videobuf2_dma_contig,videobuf2_memops,videobuf2_v4l2, Live 0x0000000000000000
mc 81920 6 rpi_hevc_dec,pisp_be,v4l2_mem2mem,videobuf2_v4l2,videodev,videobuf2_common, Live 0x0000000000000000
i2c_brcmstb 49152 0 - Live 0x0000000000000000
spi_bcm2835 49152 0 - Live 0x0000000000000000
gpio_keys 49152 0 - Live 0x0000000000000000
rp1_pio 65536 0 - Live 0x0000000000000000
pwm_fan 49152 0 - Live 0x0000000000000000
raspberrypi_gpiomem 49152 0 - Live 0x0000000000000000
rp1_adc 49152 0 - Live 0x0000000000000000
rp1_fw 49152 1 rp1_pio, Live 0x0000000000000000
rp1_mailbox 49152 1 - Live 0x0000000000000000
nvmem_rmem 49152 0 - Live 0x0000000000000000
i2c_dev 49152 0 - Live 0x0000000000000000
ledtrig_pattern 49152 0 - Live 0x0000000000000000
fuse 196608 5 - Live 0x0000000000000000
dm_mod 163840 0 - Live 0x0000000000000000
ip_tables 65536 0 - Live 0x0000000000000000
x_tables 81920 1 ip_tables, Live 0x0000000000000000
ipv6 606208 52 [permanent], Live 0x0000000000000000
vc4 425984 9 - Live 0x0000000000000000
snd_soc_hdmi_codec 49152 2 - Live 0x0000000000000000
snd_soc_core 311296 2 vc4,snd_soc_hdmi_codec, Live 0x0000000000000000
snd_compress 49152 1 snd_soc_core, Live 0x0000000000000000
snd_pcm_dmaengine 49152 1 snd_soc_core, Live 0x0000000000000000
snd_pcm 147456 4 snd_soc_hdmi_codec,snd_soc_core,snd_compress,snd_pcm_dmaengine, Live 0x0000000000000000
snd_timer 65536 3 snd_hrtimer,snd_seq,snd_pcm, Live 0x0000000000000000
snd 131072 9 snd_seq,snd_seq_device,snd_soc_hdmi_codec,snd_soc_core,snd_compress,snd_pcm,snd_timer, Live 0x0000000000000000
hci_uart 65536 0 - Live 0x0000000000000000
btbcm 49152 1 hci_uart, Live 0x0000000000000000
drm_display_helper 49152 1 vc4, Live 0x0000000000000000
bluetooth 638976 33 rfcomm,bnep,hci_uart,btbcm, Live 0x0000000000000000
libaes 49152 4 aes_ce_blk,aes_ce_cipher,ghash_ce,bluetooth, Live 0x0000000000000000
rfkill 49152 6 cfg80211,bluetooth, Live 0x0000000000000000
ecdh_generic 49152 2 bluetooth, Live 0x0000000000000000
v3d 212992 5 - Live 0x0000000000000000
drm_dma_helper 49152 1 vc4, Live 0x0000000000000000
drm_shmem_helper 49152 1 v3d, Live 0x0000000000000000
cec 65536 1 vc4, Live 0x0000000000000000
ecc 65536 1 ecdh_generic, Live 0x0000000000000000
drm_kms_helper 245760 3 vc4,drm_dma_helper,drm_shmem_helper, Live 0x0000000000000000
gpu_sched 98304 1 v3d, Live 0x0000000000000000
drm 688128 16 vc4,drm_display_helper,v3d,drm_dma_helper,drm_shmem_helper,drm_kms_helper,gpu_sched, Live 0x0000000000000000
drm_panel_orientation_quirks 65536 1 drm, Live 0x0000000000000000
backlight 49152 2 drm_kms_helper,drm, Live 0x0000000000000000
uio_pdrv_genirq 49152 0 - Live 0x0000000000000000
uio 49152 1 uio_pdrv_genirq, Live 0x0000000000000000
</details>


/proc/modules と lsmod / modprobe / insmod の関係

/proc/modules
→ 「今ロードされているモジュール一覧」を
カーネルが用意している擬似ファイル。
cat で読むだけ（ユーザ空間の通常のプログラム）。

lsmod
→ 中身はほぼ /proc/modules の見やすい表示。

insmod
→ 指定した .ko を 無理やり1個だけ カーネルに挿すツール
（依存関係の解決は自分でやる必要がある）。

modprobe
→ 「名前」を渡すと、依存関係も含めて /lib/modules 以下から
.ko を探してよしなにロードしてくれる賢いツール。
→ これは ユーザランドのプログラム で、
システムコールを使ってカーネルに「このモジュール入れて」と頼む。