**参考**

- [GPIO Pin](https://control-eng-life.com/raspberry-pi-5-pin/)
- 

## 事前準備
```/boot/firmware/config.txt```を修正して末尾に以下を追加
- rp1-i2s：Pi5でI2Sハードを有効化
- googlevoicehat-soundcard：PDMマイクをALSAカードに束ねる
```
dtparam=i2s=on
dtoverlay=rp1-i2s
dtoverlay=googlevoicehat-soundcard
```
再起動
```
sudo reboot
```

---
## PDMマイク(SPH0641)の接続
| 信号   | Raspberry Pi GPIO    | 物理ピン     |
| ---- | -------------------- | -------- |
| VDD  | 3.3V                 | Pin 1    |
| GND  | GND                  | Pin 6 など |
| CLK  | **GPIO18 (PCM_CLK)** | Pin 12   |
| DATA | **GPIO20 (PCM_DIN)** | Pin 38   |

## Linux から認識されているかの確認

```
lsmod | grep bcm
spi_bcm2835            49152  0
btbcm                  49152  1 hci_uart
bluetooth             638976  33 hci_uart,btbcm,bnep,rfcomm
```

```
arecord -l
**** List of CAPTURE Hardware Devices ****
card 2: sndrpigooglevoi [snd_rpi_googlevoicehat_soundcar], device 0: Google voiceHAT SoundCard HiFi voicehat-hifi-0 [Google voiceHAT SoundCard HiFi voicehat-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```
以下が確認できた。
- Device Tree overlay が正しく適用された
- RP1 I2S が有効化された
- PDMマイクが ALSAの録音デバイスとして登録された
- kernel → ALSA → userspace の経路が生きている

---

## デバイス仕様
48 kHz 固定
おそらくモノラル


## 動作確認

**録音**
```
arecord -D hw:2,0 -f S32_LE -r 16000 -c 2 -d 5 test.wav
```

こっちが正しいっぽい。
```
kenny@raspberrypi:~/Sound $ arecord -D hw:2,0 -f S32_LE -r 48000 -c 2 -d 5 raw_48k.wav
Recording WAVE 'raw_48k.wav' : Signed 32 bit Little Endian, Rate 48000 Hz, Stereo
```

**再生**
（SoC側で再生するのが面倒なのでホスト側にSCPして再生する方が楽）
```
aplay test.wav
```

音声信号の統計量確認
```
sox test.wav -n stat
Samples read:             80000
Length (seconds):      5.000000
Scaled by:         2147483647.0
Maximum amplitude:     0.999969
Minimum amplitude:    -1.000000
Midline amplitude:    -0.000015
Mean    norm:          0.224325
Mean    amplitude:    -0.001978
RMS     amplitude:     0.277220
Maximum delta:         1.411102
Minimum delta:         0.000000
Mean    delta:         0.269977
RMS     delta:         0.339230
Rough   frequency:         3116
Volume adjustment:        1.000
```

なぜか取得音声が砂嵐になる。確認用のコマンド。
```
kenny@raspberrypi:~/Sound $ arecord -D hw:2,0 --dump-hw-params
Warning: Some sources (like microphones) may produce inaudible results
         with 8-bit sampling. Use '-f' argument to increase resolution
         e.g. '-f S16_LE'.
Recording WAVE 'stdin' : Unsigned 8 bit, Rate 8000 Hz, Mono
HW Params of device "hw:2,0":
--------------------
ACCESS:  MMAP_INTERLEAVED RW_INTERLEAVED
FORMAT:  S32_LE
SUBFORMAT:  STD
SAMPLE_BITS: 32
FRAME_BITS: 64
CHANNELS: 2
RATE: 48000
PERIOD_TIME: (83 682667)
PERIOD_SIZE: [4 32768]
PERIOD_BYTES: [32 262144]
PERIODS: [2 16384]
BUFFER_TIME: (166 1365334)
BUFFER_SIZE: [8 65536]
BUFFER_BYTES: [64 524288]
TICK_TIME: ALL
--------------------
arecord: set_params:1352: Sample format non available
Available formats:
- S32_LE

```


結局動かなかった。購入したマイクはPDM（pulse density modulation）マイクで、出力はPDM（1MHz）のパルスデータ。それに対してRaspberry Pi5が想定しているのはPCM（pulse code modulation）であり、PDMをPCMから入力すると、それっぽいデータとしては扱えるが中身に意味はなく、砂嵐となる。

PCMで出力するマイクがあるとよい。