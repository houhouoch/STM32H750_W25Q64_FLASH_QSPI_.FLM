# STM32H750 + W25Q64 QSPI 下载算法制作指南 (FLM)

本项目是基于安富莱（Armfly）教程移植的 **STM32H750** 专用外部 QSPI Flash（W25Q64）下载算法工程。编译生成的 `.FLM` 文件可直接用于 Keil MDK 的 Flash 下载。

---

## 📂 1. 项目目录结构

* **User/**：核心算法实现。
    * `FlashDev.c`：定义 Flash 器件信息（起始地址 0x90000000、容量、扇区大小）。
    * `FlashPrg.c`：定义核心逻辑（Init、Erase、Program）。
    * `bsp/`：QSPI 底层驱动代码。
* **Project/**：MDK-ARM 工程文件。
* **Libraries/**：存放 STM32H7 HAL 库和 CMSIS 核心文件。

---

## 🛠️ 2. 移植与配置关键步骤

制作 FLM 算法不同于普通 App 开发，以下几个配置如果漏掉，算法必挂：

### 2.1 MDK 工程特殊设置 (最重要)
由于算法是下载到 RAM 运行的，必须开启 **PIC（位置无关代码）** 模式：
1.  **Options for Target -> C/C++**：勾选 `Read-Only Position Independent` 和 `Read-Write Position Independent`。
2.  **Options for Target -> Output**：将 `Name of Executable` 后缀手动改为 `.FLM`。
3.  **Options for Target -> User**：建议添加脚本在编译后自动将生成的 `.FLM` 复制到 Keil 安装目录的 `ARM/Flash` 文件夹。

### 2.2 时钟初始化逻辑
在 `FlashPrg.c` 的 `Init` 函数中，**必须使用 HSI (内部高速振荡器)**。
> **避坑提示**：下载算法运行时，目标板可能还处于刚复位的混乱状态。如果使用 HSE（外部晶振），一旦板子硬件晶振起振慢或配置不对，Keil 会直接报 "Flash Timeout" 或 "Cannot access target"。

### 2.3 算法函数实现
* **Init**: 初始化 QSPI 引脚、外设及 Flash 进入四线模式。
* **EraseSector**: 擦除指定扇区。注意传入的 `Addr` 是绝对地址，需减去基地址 `0x90000000` 得到 Flash 内部偏移。
* **ProgramPage**: 写入数据。

---

## ⚠️ 3. 核心注意事项 (容易失败的细节)

### 1. 严禁开启中断
下载算法是一个孤立的逻辑块，运行期间 **Systick 和所有外设中断必须关闭**。
* 如果你的 QSPI 驱动里有 `HAL_Delay()`，必须改成**软件死循环延时**，否则程序会卡死在等待中断的逻辑里。

### 2. D-Cache 一致性 (H7 特有)
STM32H7 带有 Cache 缓存。
* 如果算法中使用了 DMA（不推荐在算法中使用，建议轮询），必须在操作完成后调用 `SCB_InvalidateDCache_by_Addr`，否则校验（Verify）环节会读到 Cache 里的旧数据导致失败。
* **安富莱建议**：在算法初始化阶段直接关闭 D-Cache 最稳妥。

### 3. 堆栈大小设置
算法运行在 RAM 的受限空间内。在 `Project` 的汇编启动文件或链接脚本中，`Stack` 空间不要设得太大（通常 512B ~ 1KB 足够），否则可能挤占算法逻辑空间。

### 4. 喂狗逻辑
如果你的硬件开启了硬件看门狗，在 `EraseChip`（整片擦除）这种长耗时函数中，**必须在循环体内手动喂狗**，防止擦除到一半板子复位。

---

## ✅ 4. 如何验证算法

1.  编译工程，获得 `XXXX.FLM`。
2.  将文件放入 Keil 路径：`C:\Keil_v5\ARM\Flash`。
3.  在目标工程的 `Flash Download` 设置里 `Add` 这个算法。
4.  尝试点击 `Download`。如果成功下载且校验通过，说明算法移植完美。

---
/********************************** END **********************************/
