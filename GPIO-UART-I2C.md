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

```mermaid
flowchart LR
    subgraph ControlPlane["コントロールプレーン"]
        CPU["CPU / ドライバ"] --> I2C["I²C (制御信号)"]
        CPU --> GPIO["GPIO (電源/リセット)"]
    end

    subgraph DataPlane["データプレーン"]
        Sensor["📷 カメラセンサ (RAW出力)"] --> CSI["MIPI-CSI RX (高速データレーン)"]
        CSI --> ISP["ISP / GPU (画像処理ブロック)"]
    end

    ControlPlane --> DataPlane
```

```mermaid
flowchart TB
    subgraph Control["🎛 コントロールプレーン（Control Plane）"]
        CPU["CPU / カーネルドライバ"]
        GPIO["GPIO：電源ON/OFFやリセット制御"]
        I2C["I²C：センサ設定（露光時間・ゲインなど）"]
        UART["UART / SPI：通信・デバッグ"]
        CPU --> GPIO
        CPU --> I2C
        CPU --> UART
    end

    subgraph Data["🎥 データプレーン（Data Plane）"]
        Sensor["📷 カメラセンサ（RAW出力）"]
        MIPI["MIPI-CSI PHY：高速シリアル伝送"]
        ISP["ISPブロック：デモザイク / AWB / ノイズ除去"]
        GPU["GPU / メモリ（DMA経由転送）"]
        Sensor --> MIPI --> ISP --> GPU
    end

    Control -->|設定・制御| Data
```
