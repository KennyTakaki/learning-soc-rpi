**参考**

- https://www.youtube.com/watch?v=ziBRtB8E6Ps&t=193s
- https://www.indoorcorgielec.com/resources/raspberry-pi/camera-setup/
- https://zenn.dev/thorie/scraps/663b022248e67a
- https://docs.freenove.com/projects/fnk0056/en/latest/fnk0056/codes/tutorial/Get_Started.html

MIPI-CSIの接続をRaspberryPi5に認識させるためにはconfig.txtを修正する必要がある。
☆部分の修正と追加が必要。
```
kenny@raspberrypi:~ $ cat /boot/firmware/config.txt
# For more options and information see
# http://rptl.io/configtxt
# Some settings may impact device functionality. See link above for details

# Uncomment some or all of these to enable the optional hardware interfaces
#dtparam=i2c_arm=on
#dtparam=i2s=on
#dtparam=spi=on

# Enable audio (loads snd_bcm2835)
dtparam=audio=on

# Additional overlays and parameters are documented
# /boot/firmware/overlays/README

# Automatically load overlays for detected cameras
camera_auto_detect=0 ☆

# Automatically load overlays for detected DSI displays
display_auto_detect=1

# Automatically load initramfs files, if found
auto_initramfs=1

# Enable DRM VC4 V3D driver
dtoverlay=vc4-kms-v3d
max_framebuffers=2

# Don't have the firmware create an initial video= setting in cmdline.txt.
# Use the kernel's default instead.
disable_fw_kms_setup=1

# Run in 64-bit mode
arm_64bit=1

# Disable compensation for displays with overscan
disable_overscan=1

# Run as fast as firmware / board allows
arm_boost=1

[cm4]
# Enable host mode on the 2711 built-in XHCI USB controller.
# This line should be removed if the legacy DWC2 controller is required
# (e.g. for USB device mode) or if USB support is not required.
otg_mode=1

[cm5]
dtoverlay=dwc2,dr_mode=host

[all]
dtoverlay=imx219,cam0 ☆
```
---
## カメラが認識されていることを確認

```
kenny@raspberrypi:~ $ ls /dev/i2c*
/dev/i2c-10  /dev/i2c-13  /dev/i2c-14  /dev/i2c-6
```
この場合、/dev/i2c-10


lsmodでカーネルモジュールがロードされていることを確認

```
kenny@raspberrypi:~ $ lsmod | grep rp1
rp1_cfe                65536  18
v4l2_fwnode            49152  5 imx219,rp1_cfe
v4l2_async             49152  3 v4l2_fwnode,imx219,rp1_cfe
videobuf2_dma_contig    49152  3 pisp_be,rpi_hevc_dec,rp1_cfe
videobuf2_v4l2         49152  4 pisp_be,rpi_hevc_dec,rp1_cfe,v4l2_mem2mem
videodev              344064  36 v4l2_async,v4l2_fwnode,imx219,pisp_be,rpi_hevc_dec,videobuf2_v4l2,rp1_cfe,v4l2_mem2mem
videobuf2_common       98304  7 pisp_be,rpi_hevc_dec,videobuf2_dma_contig,videobuf2_v4l2,rp1_cfe,v4l2_mem2mem,videobuf2_memops
rp1_pio                65536  0
mc                     81920  13 v4l2_async,videodev,imx219,pisp_be,rpi_hevc_dec,videobuf2_v4l2,rp1_cfe,videobuf2_common,v4l2_mem2mem
rp1_mailbox            49152  1
rp1_adc                49152  0
rp1_fw                 49152  1 rp1_pio
```
v4l2がvideo for linux2なのでカメラの処理に必要なモジュールがロードされている。

imx219 が他のモジュールの依存として名前に出ている
→ センサドライバ（imx219）がロードされている
→ I²C 経由でセンサが検出できていないと、そもそもここに出てこないケースが多い

rp1_cfe
→ RP1 内の Camera Front End（CSI 受信側）ドライバ
→ これが載っていて（18 という使用カウントも付いている）、videobuf2_* や videodev, v4l2_*, mc がぶら下がっている
→ V4L2 メディアパイプラインとしてはちゃんと初期化されている ことを示す

v4l2_fwnode, v4l2_async, videodev, videobuf2_*, mc
→ これらは V4L2 / Media Controller のコア部分
→ rp1_cfe や imx219 が V4L2 デバイスとして登録されているから引っ張られてきている

つまり、
rp1＋imx219＋V4L2 のカーネル側スタックはちゃんと動いている可能性が高い


videoがデバイスとして見えているか？
```
kenny@raspberrypi:~ $ ls -l /dev/video*
crw-rw----+ 1 root video 81, 17 Nov 25 23:56 /dev/video0
crw-rw----+ 1 root video 81, 18 Nov 25 23:56 /dev/video1
crw-rw----+ 1 root video 81,  4 Nov 25 23:56 /dev/video19
crw-rw----+ 1 root video 81, 19 Nov 25 23:56 /dev/video2
crw-rw----+ 1 root video 81,  0 Nov 25 23:56 /dev/video20
crw-rw----+ 1 root video 81,  1 Nov 25 23:56 /dev/video21
crw-rw----+ 1 root video 81,  2 Nov 25 23:56 /dev/video22
crw-rw----+ 1 root video 81,  3 Nov 25 23:56 /dev/video23
crw-rw----+ 1 root video 81,  5 Nov 25 23:56 /dev/video24
crw-rw----+ 1 root video 81,  6 Nov 25 23:56 /dev/video25
crw-rw----+ 1 root video 81,  7 Nov 25 23:56 /dev/video26
crw-rw----+ 1 root video 81,  8 Nov 25 23:56 /dev/video27
crw-rw----+ 1 root video 81,  9 Nov 25 23:56 /dev/video28
crw-rw----+ 1 root video 81, 10 Nov 25 23:56 /dev/video29
crw-rw----+ 1 root video 81, 20 Nov 25 23:56 /dev/video3
crw-rw----+ 1 root video 81, 11 Nov 25 23:56 /dev/video30
crw-rw----+ 1 root video 81, 12 Nov 25 23:56 /dev/video31
crw-rw----+ 1 root video 81, 13 Nov 25 23:56 /dev/video32
crw-rw----+ 1 root video 81, 14 Nov 25 23:56 /dev/video33
crw-rw----+ 1 root video 81, 15 Nov 25 23:56 /dev/video34
crw-rw----+ 1 root video 81, 16 Nov 25 23:56 /dev/video35
crw-rw----+ 1 root video 81, 21 Nov 25 23:56 /dev/video4
crw-rw----+ 1 root video 81, 22 Nov 25 23:56 /dev/video5
crw-rw----+ 1 root video 81, 23 Nov 25 23:56 /dev/video6
crw-rw----+ 1 root video 81, 24 Nov 25 23:56 /dev/video7
```
見えているが、なぜこんなに多いか？
→
RP1＋Pi カーネルだと、1個のカメラだけでも
CSI の入力（rp1_cfe / capture ノード）
ISP / スケーラー
コーデック（H.264/HEVC の mem2mem デバイス）
そのほか内部用のノード
などがそれぞれ /dev/videoX として複数個作られるようだ。


ripcam-helloコマンドで確認してみる。
```
kenny@raspberrypi:~ $ rpicam-hello --list-cameras
Available cameras
-----------------
0 : imx219 [3280x2464 10-bit RGGB] (/base/axi/pcie@1000120000/rp1/i2c@88000/imx219@10)
    Modes: 'SRGGB10_CSI2P' : 640x480 [103.33 fps - (1000, 752)/1280x960 crop]
                             1640x1232 [41.85 fps - (0, 0)/3280x2464 crop]
                             1920x1080 [47.57 fps - (680, 692)/1920x1080 crop]
                             3280x2464 [21.19 fps - (0, 0)/3280x2464 crop]
           'SRGGB8' : 640x480 [103.33 fps - (1000, 752)/1280x960 crop]
                      1640x1232 [41.85 fps - (0, 0)/3280x2464 crop]
                      1920x1080 [47.57 fps - (680, 692)/1920x1080 crop]
                      3280x2464 [21.19 fps - (0, 0)/3280x2464 crop]
```
認識されていて、ユーザランドからカメラとして利用可能な状態になっているようだ。

---
## 動作確認
```
# プレビュー（ウィンドウ表示）
rpicam-hello

# 静止画撮影
rpicam-still -o test.jpg

# 動画撮影（5秒）
rpicam-vid -o test.h264 -t 5000
```

**コマンドでの動画再生**

```
sudo apt update
sudo apt install mpv

再生
mpv test.h264

全画面再生
mpv --fs test.mp4

音声なし
mpv --no-audio test.h264 
```
