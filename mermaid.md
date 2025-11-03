flowchart TB
    subgraph SOC["Broadcom BCM2712（Application Processor, AP）"]
        A76[CPU: Quad Cortex-A76<br/>Main OS (Linux) 実行]
        GPU[GPU: VideoCore VII]
        MEM[メモリコントローラ<br/>LPDDR4X]
        PCIe[PCIe 2.0 ×4 インターフェース]
    end

    subgraph RP1["RP1 Peripheral Controller（サウスブリッジ）"]
        subgraph M3["Cortex-M3 ×2<br/>(制御用マイクロコントローラ)"]
            ROM[Boot ROM]
            SRAM[Shared SRAM (64KB)]
        end
        USB[USB3/USB2 Host Controllers]
        ETH[Ethernet MAC<br/>(RGMII経由で外部PHYへ)]
        GPIO[GPIO (28ピン)]
        LSIO[Low-speed I/O<br/>(I²C / SPI / UART / I²S / PWM)]
        MIPI[MIPI CSI-2 / DSI<br/>Camera & Display]
        ADC[ADC (アナログ入力＋温度センサ)]
        DMA[DMA Controller (8ch)]
        PLL[PLL & Clock Generators]
    end

    STORAGE[microSDカードスロット]
    PHY[Gigabit Ethernet PHY]
    USB_PORTS[USB3.0 / USB2.0 ポート]
    CAM[カメラモジュール]
    DISP[ディスプレイコネクタ]
    GPIO_PIN[40ピン GPIOヘッダ]

    %% Connections
    A76 --> PCIe
    PCIe --> RP1
    RP1 --> USB_PORTS
    RP1 --> PHY
    RP1 --> GPIO_PIN
    RP1 --> CAM
    RP1 --> DISP
    RP1 --> STORAGE

    PHY -->|RGMII| ETH