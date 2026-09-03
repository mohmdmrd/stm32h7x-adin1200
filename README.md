# STM32H7x ADIN1200 Ethernet Driver

ADIN1200 PHY driver for STM32H7x series MCUs.  
⚠️ **Note:** This driver is only tested on **STM32H7x**.  
STM32F4 series is **not supported**.  

Since STM32CubeMX only provides support for **LAN8742**, we must manually modify the generated LwIP and LAN8742 driver files to work with **ADIN1200**.

---

## 🔧 Initialization (STM32CubeMX Configuration)

1. Enable **Ethernet** and verify MII/RMII pin configuration according to your schematic.  
2. Enable **LwIP**:  
   - Set a **static IP**  
   - Enable the **LAN8742 driver** (we will later replace this in code).  
3. Enable **FreeRTOS v2**.  
4. Enable **Ethernet interrupt** and set priority to **15** (lowest priority).

---

## 🧩 Cortex-M7 MPU Configuration

You must configure MPU regions for **DMA buffers** and **LwIP heap**:

- `0x30000000` → Ethernet RX/TX DMA buffer  
- `0x30004000` → `LWIP_RAM_HEAP_POINTER`

Example (CubeIDE MPU config screenshots):

<img width="707" height="361" alt="image" src="https://github.com/user-attachments/assets/2e75acfd-e8c6-4b9e-86b7-b605e308517e" />
<img width="690" height="437" alt="image" src="https://github.com/user-attachments/assets/58592a42-5ad5-470a-b2a3-70c94adb4cae" />

If using **external SRAM**, add its address section inside `STM32xx_FLASH.ld`:

```c
.lwip_sec (NOLOAD) : {
    . = ABSOLUTE(0x30000000);
    *(.RxDecripSection) 
    
    . = ABSOLUTE(0x30000080);
    *(.TxDecripSection)
    
    . = ABSOLUTE(0x30000100);
    *(.Rx_PoolSection) 
  } >RAM_D2

.ARM.attributes 0 : { *(.ARM.attributes) }
```

---

## 📝 Code Modification for ADIN1200

1. Copy the following files into your project **root folder** (next to `Drivers/`):  
   - `adin1200.exe`  
   - `adin1200.dll`  
   - `adin1200.deps.json`  
   - `adin1200.pdb`  
   - `adin1200.runtimeconfig.json`  

2. Run `adin1200.exe`.  
   - If you see `successfully!`, it means the libraries were patched correctly and you can use ADIN1200.

---

## ✅ Testing (Ping)

1. In `main.c`, add a small **delay (e.g., 1s)** **before `MX_LWIP_Init();`**.  
   - Reason: **STM32H7 must power up after ADIN1200.**  
   - Do **not** use `osDelay()` inside LwIP tasks.  

2. Flash the firmware, connect Ethernet, and open **CMD** (Windows) or **Terminal** (Linux/Mac):  

```bash
ping 192.168.1.xxx
```

If everything is configured properly, you should receive a ping response 🎉.
