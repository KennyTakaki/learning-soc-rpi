```mermaid
flowchart LR
    CPU["CPU (A76コア)"] -->|命令| Kernel["Linuxカーネル<br/>(GPIO/UART/I2Cドライバ)"]
    Kernel --> MMIO["MMIOレジスタ<br/>(SoC内部I/O制御ブロック)"]
    MMIO --> PAD["物理ピン<br/>(GPIOヘッダ)"]
    PAD --> DEVICE["LED・センサ・外部マイコン等"]
```
