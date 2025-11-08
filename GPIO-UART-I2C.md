```mermaid
flowchart LR
    CPU["CPU (A76コア)"] -->|命令| Kernel["Linuxカーネル<br/>(GPIO/UART/I2Cドライバ)"]
    Kernel --> MMIO["MMIOレジスタ<br/>(SoC内部I/O制御ブロック)"]
    MMIO --> PAD["物理ピン<br/>(GPIOヘッダ)"]
    PAD --> DEVICE["LED・センサ・外部マイコン等"]
```
CPUは直接ピンを操作できません。

Linuxの **ドライバ（GPIOサブシステム）** がMMIO（Memory-Mapped I/O）経由でSoCのレジスタを書き換え、
その結果、物理ピンに電圧が出たり消えたりします。

Linuxでは、ハードウェアもファイルとして抽象化されています。
つまり /sys/class/gpio/gpio18/value に “1” を書くのは、
「このピンに高電圧（HIGH）を出せ」という指令です。

Raspberry PiのSoCピンは、1本の線が複数の機能を持ちます。  
```mermaid
flowchart TD
    Pin["物理ピン (例: GPIO14)"] -->|Mode0| UART_TX
    Pin -->|Mode1| GPIO_OUT
    Pin -->|Mode2| PWM

```
