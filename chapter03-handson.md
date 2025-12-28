# 最終的な確認コマンド

```
arecord -l
arecord -D hw:2,0 -c 2 -r 48000 -f S32_LE -t wav -d 10 test.wav
```

# 利用マイク
I2S INMP441

# Raspberry Pi5 のI2S端子
| 信号             | GPIO   | 役割              |
| -------------- | ------ | --------------- |
| **BCLK**       | GPIO18 | ビットクロック         |
| **LRCLK (WS)** | GPIO19 | L/R切替           |
| **DIN**        | GPIO20 | マイク → Pi        |
| **DOUT**       | GPIO21 | Pi → DAC（今回は不要） |


# 今回の配線

| マイク            | Raspberry Pi 5 |
| -------------- | -------------- |
| **VDD / VCC**  | 3.3V           |
| **GND**        | GND            |
| **SCK / BCLK** | GPIO18         |
| **WS / LRCLK** | GPIO19         |
| **SD / DOUT**  | GPIO20         |
| **L/R or SEL** | GND (or 3.3V)    |

# overlayセッティング



```
/boot/firmware/config.txt

[all]
dtoverlay=imx219,cam0
dtparam=i2s=on
dtoverlay=googlevoicehat-soundcard
```

これで録音可能になる。