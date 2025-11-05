システム全体図

```mermaid
graph TD
    AP["BCM2712 (AP / 4x Cortex-A76)"]
    MEM["LPDDR4X SDRAM"]
    GPU["VideoCore VII GPU"]
    HDMI["HDMI 0/1 Output"]
    NVMe["NVMe (PCIe M.2)"]
    RP1["RP1 Peripheral Controller"]
    USB["USB 3.0 / 2.0 Ports"]
    ETH["Gigabit Ethernet"]
    MIPI["MIPI CSI / DSI"]
    GPIO["GPIO Header (SPI / I2C / UART / PWM / ADC)"]
    USD["microSD (SDIO)"]

    AP --> MEM
    AP --> GPU
    GPU --> HDMI
    AP --> NVMe
    AP -- "PCIe 2.0 x1" --> RP1
    RP1 --> USB
    RP1 --> ETH
    RP1 --> MIPI
    RP1 --> GPIO
    RP1 --> USD
```

RP1内部ブロック図
```mermaid
graph TD
    PCIeEP["PCIe Endpoint Controller"]
    BUS["Internal Interconnect / Bus"]
    USB3["USB3 Controllers"]
    ETHMAC["Ethernet MAC"]
    MIPI2["MIPI CSI / DSI"]
    SDIO["SDIO Interface"]
    GPIO2["GPIO"]
    SPI2["SPI"]
    I2C2["I2C"]
    UART2["UART"]
    PWM2["PWM"]
    ADC2["ADC"]
    DMA["DMA Controller"]
    SSRAM["Shared SRAM (64KB)"]
    LSRAM["Local SRAM"]
    BOOT["Boot ROM"]
    CLK["Clock / Reset Control"]

    PCIeEP --> BUS
    BUS --> USB3
    BUS --> ETHMAC
    BUS --> MIPI2
    BUS --> SDIO
    BUS --> GPIO2
    BUS --> SPI2
    BUS --> I2C2
    BUS --> UART2
    BUS --> PWM2
    BUS --> ADC2
    BUS --> DMA
    DMA --> SSRAM
    DMA --> LSRAM
    BOOT --> BUS
    CLK --> BUS



```

AP(BCM2712)内部ブロック図

```mermaid
flowchart TB
  subgraph CLUSTER["BCM2712 Core Cluster"]
    C0["Cortex-A76 Core 0\nL1I + L1D"] --> L2_0["L2 (Private)"]
    C1["Cortex-A76 Core 1\nL1I + L1D"] --> L2_1["L2 (Private)"]
    C2["Cortex-A76 Core 2\nL1I + L1D"] --> L2_2["L2 (Private)"]
    C3["Cortex-A76 Core 3\nL1I + L1D"] --> L2_3["L2 (Private)"]
  end

  L3S["L3 (Shared)"]
  FAB["System Fabric (AXI / NoC)"]
  MEMC["LPDDR4X Memory Controller"]
  RAM["LPDDR4X SDRAM"]
  VCGPU["VideoCore VII GPU"]
  DISP["Display Controller"]
  PCIE["PCIe 2.0 Controller"]

  L2_0 --> L3S
  L2_1 --> L3S
  L2_2 --> L3S
  L2_3 --> L3S

  L3S --> FAB
  VCGPU --> FAB
  PCIE --> FAB
  FAB --> MEMC
  MEMC --> RAM
  VCGPU --> DISP
```

AP(BCM2712)内部ブロック図
```mermaid
flowchart TB
  subgraph Core["Cortex-A76 Core"]
    L1I["L1 Instruction Cache (64KB)"]
    L1D["L1 Data Cache (64KB)"]
    REG["Registers / Pipeline"]
    REG --> L1I
    REG --> L1D
  end

  L2["L2 Cache (Per-Core, 512KB)"]
  L3["L3 Cache (Shared)"]
  MEM["LPDDR4X Memory"]

  Core --> L2
  L2 --> L3
  L3 --> MEM
```



```mermaid
flowchart LR
    subgraph PCIeBus["PCIe Bus (BCM2712 ⇄ RP1)"]
        PCIe_EP["PCIe Endpoint Controller"]
    end

    subgraph RP1_Interconnect["RP1内部インターコネクト"]
        direction TB
        subgraph AXI["AXI (高速バス)"]
            USB3["USB3 Controllers"]
            DMA["DMA Controller"]
            ETH["Ethernet MAC"]
            MIPI["MIPI CSI/DSI"]
        end
        subgraph AHB["AHB (中速バス)"]
            SDIO["SDIO Interface"]
        end
        subgraph APB["APB (低速バス)"]
            GPIO["GPIO"]
            SPI["SPI"]
            I2C["I²C"]
            UART["UART"]
            I2S["I²S"]
            PWM["PWM"]
            ADC["ADC"]
            CLKS["Clock/Reset Control"]
        end
    end

    subgraph CTRL["Cortex-M3 Dual Cluster"]
        ROM["Boot ROM"]
        ISRAM["Local ISRAM/DSRAM"]
        SHRAM["Shared SRAM (64KB)"]
    end

    PCIe_EP --> AXI
    AXI --> AHB
    AHB --> APB
    CTRL --> APB
    CTRL --> SHRAM
```

dig2

```mermaid
flowchart TB
  subgraph AP["BCM2712 (Application Processor)"]
    A76["CPU: Cortex-A76 x4\nMain OS: Linux"]
    GPU["GPU: VideoCore VII"]
    MEM["Memory Ctrl: LPDDR4X"]
    RC["PCIe 2.0 x4 Root Complex"]
    SDIO["SDIO for microSD"]
  end

  subgraph RP1["RP1 Peripheral Controller (Southbridge)"]
    M3["Dual Cortex-M3\nBoot ROM + 64KB SRAM"]
    USB["USB3/USB2 Host Controllers"]
    ETH["Ethernet MAC (to ext PHY via RGMII)"]
    GPIO["GPIO 28 pins"]
    LSIO["I2C / SPI / UART / I2S / PWM"]
    MIPI["MIPI CSI-2 / DSI (Camera & Display)"]
    ADC["ADC + Temp sensor"]
    DMA["DMA 8 channels"]
    PLL["PLL & Clock Generators"]
  end

  STORAGE["microSD slot"]
  PHY["Gigabit Ethernet PHY"]
  USBP["USB3.0 / USB2.0 Ports"]
  CAM["Camera module"]
  DISP["Display connector"]
  GPIOHDR["40-pin GPIO header"]

  %% Connections
  A76 --> RC
  RC --> RP1
  SDIO --> STORAGE
  RP1 --> USBP
  RP1 --> PHY
  RP1 --> GPIOHDR
  RP1 --> CAM
  RP1 --> DISP
  PHY -->|RGMII| ETH
```

④ 通信系統図（データフロー／バス構造図）
```mermaid
flowchart LR;
subgraph AP["BCM2712 AP"];
CPU1["Cortex-A76 x4"];
GPU1["VideoCore VII"];
MEM1["LPDDR4X SDRAM"];
AXI1["AXI / AHB fabric"];
end;

subgraph RP1["RP1 peripheral controller"];
PCIE1["PCIe controller Gen2 x4"];
USB3H["USB 3.0 x2 (5 Gbps)"];
USB2H["USB 2.0 x2 (480 Mbps)"];
ETH1["Gigabit Ethernet MAC (1 Gbps)"];
CSI1["MIPI CSI-2"];
DSI1["MIPI DSI"];
SDIO1["microSD SDR104 (104 MBps)"];
end;

subgraph EXT["External peripherals"];
CAM["Camera module"];
DISP["Display panel"];
USBDEV["USB devices / storage"];
LAN["LAN cable"];
SDC["SD card"];
end;

CPU1<-->AXI1;
GPU1<-->AXI1;
MEM1<-->AXI1;

AXI1-- "PCIe 2.0 x4 (~16-20 Gbps)" ---PCIE1;

PCIE1-->USB3H;
PCIE1-->USB2H;
PCIE1-->ETH1;
PCIE1-->CSI1;
PCIE1-->DSI1;
PCIE1-->SDIO1;

USB3H-->USBDEV;
USB2H-->USBDEV;
ETH1-->LAN;
CSI1-->CAM;
DSI1-->DISP;
SDIO1-->SDC;

USB3H-. "DMA" .->MEM1;
ETH1-. "DMA" .->MEM1;
SDIO1-. "DMA" .->MEM1;
CSI1-. "DMA" .->MEM1;
GPU1-. "framebuffer DMA" .->MEM1;
```

⑤ コアレベル図（Cortex-A76 / Cortex-M0+ など）  
Cortex-A76 パイプラインとキャッシュ階層
```mermaid
flowchart LR
  %% ===== Cortex-A76: 命令処理とメモリ階層 =====
  subgraph A76["Cortex-A76 Core"]
    F[Fetch]
    D[Decode and Rename]
    IS[Dispatch and Issue]
    EX[Execute: ALU・FP・LoadStore]
    CM[Commit and Retire]
    IRQI[CPU Interface for IRQ]
    F --> D --> IS --> EX --> CM
  end

  subgraph Private["Private Caches per core"]
    L1I[L1I Instruction Cache]
    L1D[L1D Data Cache]
    L2[L2 Cache]
  end

  %% 参照経路（点線）
  F -. instruction .-> L1I
  EX -. data .-> L1D

  %% キャッシュ階層と一貫性
  L1I <--> L2
  L1D <--> L2
  L2 <--> CC[Coherent Interconnect]
  CC <--> SC[System Cache or L3]
  CC <--> MEM[Main Memory]
  CC <--> IO[Other Coherent Agents]

  %% 割り込み経路
  GIC[GIC Distributor and CPU Interface] -->|IRQ| IRQI

  %% 注釈ノード（点線で紐づけ）
  NOTE[Coherency is kept by snoop via interconnect]:::note
  CC -. info .- NOTE

  classDef note fill:#fff,stroke:#999,stroke-dasharray:3 3,color:#333;
```
Aコア × Mコア 役割分担と通信
```mermaid
flowchart LR
  %% ===== Cortex-A76: 命令処理とメモリ階層 =====
  subgraph A76["Cortex-A76 Core"]
    F[Fetch]
    D[Decode and Rename]
    IS[Dispatch and Issue]
    EX[Execute: ALU・FP・LoadStore]
    CM[Commit and Retire]
    IRQI[CPU Interface for IRQ]
    F --> D --> IS --> EX --> CM
  end

  subgraph Private["Private Caches per core"]
    L1I[L1I Instruction Cache]
    L1D[L1D Data Cache]
    L2[L2 Cache]
  end

  %% 参照経路（点線）
  F -. instruction .-> L1I
  EX -. data .-> L1D

  %% キャッシュ階層と一貫性
  L1I <--> L2
  L1D <--> L2
  L2 <--> CC[Coherent Interconnect]
  CC <--> SC[System Cache or L3]
  CC <--> MEM[Main Memory]
  CC <--> IO[Other Coherent Agents]

  %% 割り込み経路
  GIC[GIC Distributor and CPU Interface] -->|IRQ| IRQI

  %% 注釈ノード（点線で紐づけ）
  NOTE[Coherency is kept by snoop via interconnect]:::note
  CC -. info .- NOTE

  classDef note fill:#fff,stroke:#999,stroke-dasharray:3 3,color:#333;
```

⑥ 電源・クロック・リセット構成図（Power/Clock Tree）汎用
```mermaid
flowchart TB
  %% =========================
  %%  電源・クロック・リセット構成図（Power / Clock Tree）
  %%  目的: 起動順序と電力制御の可視化
  %% =========================

  %% ---- PMIC と電源レール ----
  subgraph PMIC[PMIC（Power Management IC / 電源シーケンサ）]
    VIN[入力電源]
    PMIC_CTRL[PMIC制御・シーケンス]
    LDO_AON[LDO（常時供給: AON）]
    BUCK_CORE[BUCK（Core レール）]
    BUCK_GPU[BUCK（GPU レール）]
    BUCK_DDR[BUCK（DDR レール）]
    BUCK_IO[BUCK（IO レール）]
    VIN --> PMIC_CTRL
    PMIC_CTRL --> LDO_AON
    PMIC_CTRL --> BUCK_CORE
    PMIC_CTRL --> BUCK_GPU
    PMIC_CTRL --> BUCK_DDR
    PMIC_CTRL --> BUCK_IO
  end

  LDO_AON --> AON_VDD[AON 電源レール]
  BUCK_CORE --> CORE_VDD[Core 電源レール]
  BUCK_GPU --> GPU_VDD[GPU 電源レール]
  BUCK_DDR --> DDR_VDD[DDR 電源レール]
  BUCK_IO  --> IO_VDD[IO 電源レール]

  %% ---- Power Good モニタ ----
  subgraph PGOOD[Power-Good モニタ]
    PGOOD_AON[PGOOD AON]
    PGOOD_CORE[PGOOD Core]
    PGOOD_GPU[PGOOD GPU]
    PGOOD_DDR[PGOOD DDR]
    PGOOD_IO[PGOOD IO]
  end

  AON_VDD --> PGOOD_AON
  CORE_VDD --> PGOOD_CORE
  GPU_VDD  --> PGOOD_GPU
  DDR_VDD  --> PGOOD_DDR
  IO_VDD   --> PGOOD_IO

  %% ---- リセット／起動シーケンス ----
  subgraph RESET[リセット / 起動シーケンス]
    POR[POR（Power-On Reset）]
    EXT_RST[外部/ウォッチドッグリセット]
    RST_CTRL[リセットコントローラ]
    BOOTROM[Boot ROM（AON 実行）]
    FW[初期ファームウェア（PM/Clock 設定）]
  end

  PGOOD_AON --> POR
  POR --> RST_CTRL
  EXT_RST --> RST_CTRL
  PGOOD_CORE --> RST_CTRL
  PGOOD_GPU  --> RST_CTRL
  PGOOD_DDR  --> RST_CTRL
  PGOOD_IO   --> RST_CTRL
  RST_CTRL --> BOOTROM

  %% ---- クロックツリー ----
  subgraph CLOCK[クロックツリー]
    XO[外部水晶発振器（基準クロック）]
    PLL_MAIN[PLL 群（主PLL・周波数合成）]
    CLK_CTRL[クロックコントローラ（分配/ゲーティング/DVFS）]
    XO --> PLL_MAIN
    PLL_MAIN --> CLK_CTRL
  end

  %% ---- 電源/クロックドメイン ----
  subgraph DOMAINS[電源/クロックドメイン]
    subgraph AON_DOM[AON（常時動作ドメイン）]
      RTC[RTC/タイマ]
      PMU[PMU（電源/電圧/状態管理）]
    end
    subgraph CORE_DOM[Core ドメイン]
      CPU[CPU クラスタ]
      INTERCONNECT[内部インターコネクト/L2]
    end
    subgraph GPU_DOM[GPU ドメイン]
      GPU[GPU]
    end
    subgraph DDR_DOM[DDR/メモリ]
      DDR_PHY[DDR PHY]
      MEM_CTRL[メモリコントローラ]
    end
    subgraph IO_DOM[IO ドメイン]
      PERI[周辺（USB/UART/SPI/I2C 等）]
    end
  end

  %% ---- AON での最初の起動（Boot ROM フェーズ） ----
  AON_VDD --> AON_DOM
  CLK_CTRL --> AON_DOM
  RST_CTRL --> AON_DOM
  BOOTROM --> PMU

  %% ---- PMU による電源投入・クロック配布・リセット解除の順次制御 ----
  PMU --> CORE_VDD
  PMU --> DDR_VDD
  PMU --> IO_VDD
  PMU --> GPU_VDD

  CLK_CTRL --> CORE_DOM
  CLK_CTRL --> DDR_DOM
  CLK_CTRL --> IO_DOM
  CLK_CTRL --> GPU_DOM

  RST_CTRL --> CORE_DOM
  RST_CTRL --> DDR_DOM
  RST_CTRL --> IO_DOM
  RST_CTRL --> GPU_DOM

  %% ---- 起動後の初期FWによる最適化/低電力 ----
  CORE_DOM --> FW
  FW --> CLK_CTRL
  FW --> PMIC_CTRL

  %% ---- 低電力状態の例 ----
  subgraph LOWPWR[低電力状態の例]
    SLEEP[Sleep：一部クロック停止（AONのみ動作）]
    DEEPSLEEP[Deep Sleep：パワーゲート＋状態保持（必要部のみ）]
  end
  FW --> SLEEP
  FW --> DEEPSLEEP

```

⑥ 電源・クロック・リセット構成図（Power/Clock Tree） RaspberryPi

```mermaid
flowchart TB
  %% =========================
  %% Raspberry Pi 5 Power / Clock / Reset (BCM2712 + RP1)
  %% 目的: 起動順序と電力制御の可視化
  %% 方針: GitHubで崩れにくい表記、HTMLタグ非使用
  %% =========================

  %% ---- PMIC と電源レール ----
  subgraph PMIC[PMIC 電源シーケンサ]
    VIN[入力電源]
    PMIC_CTRL[PMIC 制御]
    LDO_AON[AON 常時電源]
    BUCK_CPU[VDD CPU]
    BUCK_GPU[VDD GPU]
    BUCK_CORE[VDD Core SoC]
    BUCK_DDR[VDD DDR]
    BUCK_IO[VDD IO 1v8 など]
    BUCK_RP1[VDD RP1 3v3 1v8]
    VIN --> PMIC_CTRL
    PMIC_CTRL --> LDO_AON
    PMIC_CTRL --> BUCK_CPU
    PMIC_CTRL --> BUCK_GPU
    PMIC_CTRL --> BUCK_CORE
    PMIC_CTRL --> BUCK_DDR
    PMIC_CTRL --> BUCK_IO
    PMIC_CTRL --> BUCK_RP1
  end

  LDO_AON --> AON_VDD[AON 電源]
  BUCK_CPU --> CPU_VDD[CPU 電源]
  BUCK_GPU --> GPU_VDD[GPU 電源]
  BUCK_CORE --> CORE_VDD[Core 電源]
  BUCK_DDR --> DDR_VDD[DDR 電源]
  BUCK_IO  --> IO_VDD[IO 電源]
  BUCK_RP1 --> RP1_VDD[RP1 電源]

  %% ---- Power Good ----
  subgraph PGOOD[Power Good モニタ]
    PGOOD_AON[PG AON]
    PGOOD_CPU[PG CPU]
    PGOOD_GPU[PG GPU]
    PGOOD_CORE[PG Core]
    PGOOD_DDR[PG DDR]
    PGOOD_IO[PG IO]
    PGOOD_RP1[PG RP1]
  end

  AON_VDD --> PGOOD_AON
  CPU_VDD --> PGOOD_CPU
  GPU_VDD --> PGOOD_GPU
  CORE_VDD --> PGOOD_CORE
  DDR_VDD --> PGOOD_DDR
  IO_VDD  --> PGOOD_IO
  RP1_VDD --> PGOOD_RP1

  %% ---- リセットとブート経路 ----
  subgraph RESET[Reset と Boot 経路]
    POR[Power On Reset]
    EXT_RST[外部やWDTのリセット]
    RST_CTRL[リセットコントローラ]
    BOOTROM[Boot ROM]
    FW[初期FWとブートローダ]
  end

  PGOOD_AON --> POR
  PGOOD_CORE --> POR
  POR --> RST_CTRL
  EXT_RST --> RST_CTRL
  RST_CTRL --> BOOTROM

  %% ---- クロックツリー ----
  subgraph CLOCK[クロックツリー]
    XO[外部基準発振器]
    PLL_ARM[PLL ARM]
    PLL_GPU[PLL GPU]
    PLL_CORE[PLL Core]
    PLL_DDR[PLL DDR]
    CLK_CTRL[クロック分配とゲーティング]
    XO --> PLL_ARM
    XO --> PLL_GPU
    XO --> PLL_CORE
    XO --> PLL_DDR
    PLL_ARM --> CLK_CTRL
    PLL_GPU --> CLK_CTRL
    PLL_CORE --> CLK_CTRL
    PLL_DDR --> CLK_CTRL
  end

  %% ---- SoC と周辺ドメイン ----
  subgraph BCM2712[BCM2712 Application Processor]
    subgraph AON_DOM[AON 常時動作]
      RTC[RTC Timer]
      PMU[SoC 内 PMU]
    end
    subgraph CPU_DOM[CPU ドメイン]
      CPU[A76 CPU Cluster]
      INTERCONNECT[内部インターコネクト]
    end
    subgraph GPU_DOM[GPU ドメイン]
      GPU[VideoCore VII GPU]
    end
    subgraph MEM_DOM[メモリ ドメイン]
      DDR_PHY[DDR PHY]
      MEMC[メモリコントローラ]
    end
    subgraph IO_DOM[SoC 内 IO]
      SIP[セキュア初期化など]
    end
  end

  subgraph RP1[RP1 Peripheral Controller]
    RP1_PLL[RP1 PLL]
    RP1_PERI[USB Ethernet PCIe Bridge 等]
  end

  %% ---- 電源の接続 ----
  AON_VDD --> AON_DOM
  CORE_VDD --> CPU_DOM
  CORE_VDD --> IO_DOM
  GPU_VDD --> GPU_DOM
  DDR_VDD --> MEM_DOM
  RP1_VDD --> RP1

  %% ---- クロック配布 ----
  CLK_CTRL --> AON_DOM
  CLK_CTRL --> CPU_DOM
  CLK_CTRL --> GPU_DOM
  CLK_CTRL --> MEM_DOM
  CLK_CTRL --> IO_DOM
  RP1_PLL --> RP1_PERI

  %% ---- リセット配布 ----
  RST_CTRL --> AON_DOM
  RST_CTRL --> CPU_DOM
  RST_CTRL --> GPU_DOM
  RST_CTRL --> MEM_DOM
  RST_CTRL --> IO_DOM
  RST_CTRL --> RP1

  %% ---- 起動順序と相互作用 ----
  BOOTROM --> PMU
  PMU --> CPU_VDD
  PMU --> DDR_VDD
  PMU --> GPU_VDD
  PMU --> IO_VDD
  PMU --> RP1_VDD

  BOOTROM --> FW
  FW --> CLK_CTRL
  FW --> PMIC_CTRL

  %% ---- PCIe リンク確立で RP1 有効化 ----
  subgraph LINK[RP1 連携]
    PCIE_LINK[PCIe Link Up]
  end
  CPU_DOM --> PCIE_LINK
  PCIE_LINK --> RP1_PERI

  %% ---- 低電力制御例 ----
  subgraph LOWPWR[低電力状態]
    CPU_IDLE[CPU クロックゲート]
    GPU_OFF[GPU ドメイン停止]
    DDR_RET[DDR リテンション]
    SOC_DS[Deep Sleep]
  end
  FW --> CPU_IDLE
  FW --> GPU_OFF
  FW --> DDR_RET
  FW --> SOC_DS

```
⑦ メモリマップ／アドレス空間図  
AP（BCM2712）から見たメモリマップ（概要）
```mermaid
flowchart TB
  subgraph AP["AP 物理アドレス空間（概念図 / BCM2712 視点）"]
    DRAM["DRAM 領域（カーネル/ユーザ空間）"]
    RP1BAR1["RP1 Peripherals 窓（PCIe BAR1）\n例: 物理ベース 0x1F00000000 付近"]
    RP1BAR2["RP1 Shared SRAM 窓（PCIe BAR2）\n例: ベース + 0x00400000 付近"]
    APLOCAL["AP ローカル MMIO（SoC 内蔵タイマ等）"]
    BOOTROM["BootROM / 初期化領域（SoC 依存・読取専用）"]
  end

  %% 自己ループは「ここに直接アクセスする」ことのイメージ用
  DRAM --> DRAM
  RP1BAR1 --> RP1BAR1
  RP1BAR2 --> RP1BAR2
  APLOCAL --> APLOCAL
  BOOTROM --> BOOTROM

  NOTE1(("注記: RP1 内部は 0x40000000 をベースとする\n『オフセット表』で定義される"))
  RP1BAR1 -. RP1 内部は BAR1 ベース + オフセットで見える .-> NOTE1

```

RP1 ペリフェラルのオフセット（Device Tree の reg に対応）
```mermaid
flowchart TB
  subgraph RP1MAP["RP1 内部ペリフェラル（オフセット表）\nベース = 0x40000000"]
    SYSINFO["sysinfo @ 0x40000000"]
    CLOCKS["clocks_main @ 0x40018000\nclocks_video @ 0x4001C000"]
    UARTS["uart0..5 @ 0x40030000 + n*0x4000\n例: uart0 = 0x40030000"]
    SPIS["spi0.. @ 0x40050000 など"]
    I2CS["i2c0.. @ 0x40070000 など"]
    PWM["pwm0 @ 0x40098000\npwm1 @ 0x4009C000"]
    TIMER["timer @ 0x400AC000"]
    IO_BANKS["io_bank0/1/2 @ 0x400D0000/400D4000/400D8000\npads_bank0/1/2 @ 0x400F0000/400F4000/400F8000"]
    ETH["eth @ 0x40100000\neth_cfg @ 0x40104000"]
    PCIE["pcie_cfg @ 0x40108000"]
    SDIO["sdio0 @ 0x40180000\nsdio1 @ 0x40184000"]
    DMA["dma @ 0x40188000"]
    USBH["usbhost0 @ 0x40200000\nusbhost1 @ 0x40300000"]
    MIPI0["mipi0_* @ 0x40110000 付近"]
    MIPI1["mipi1_* @ 0x40128000 付近"]
    WDT["watchdog @ 0x40154000"]
    SHSRAM["Shared SRAM（RP1 ローカル）\nProcessor View: 0x20000000\nAP からは BAR2 窓経由"]
  end

  NOTE2(("対応関係の例:\nAP物理 = BAR1ベース + オフセット\n例: io_bank0 → 0x1F00000000 + 0x400D0000 = 0x1F000D0000"))
  RP1MAP -. RP1 オフセット → AP 物理 .-> NOTE2
```
