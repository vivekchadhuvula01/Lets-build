---
layout: post
title: "UART Communication with STM32 Nucleo: Receive and Display Text in PuTTY"
subtitle: "A beginner's guide to serial communication using STM32F401RE"
tags: [stm32, uart, serial, hal, putty, embedded c]
author: "Vivek Chadhuvula"
comments: true
mathjax: false
---

{: .box-success}
This tutorial will teach you how to **receive text from your STM32 MCU** and **display it in PuTTY** using the **STM32F401RE Nucleo board**. We'll use the built-in USB-to-UART converter, so no extra hardware is needed!

---

##  What is UART Communication?

{: .box-note}
**UART (Universal Asynchronous Receiver Transmitter)** is a serial communication protocol that allows your microcontroller to exchange data with other devices like PCs, GPS modules, or Bluetooth adapters.  
It transmits data bit-by-bit over two wires: **TX (Transmit)** and **RX (Receive)**.

---

##  Required Tools and Setup

| Tool | Purpose | Download/Info |
|:-----|:--------|:--------------|
| STM32CubeIDE | Development environment | [Download here](https://www.st.com/en/development-tools/stm32cubeide.html) |
| STM32CubeMX | Peripheral configuration | [Download here](https://www.st.com/en/development-tools/stm32cubemx.html) |
| PuTTY | Serial terminal application | [Download here](https://www.putty.org/) |
| STM32F401RE Nucleo | Development board | [Product link](https://www.st.com/en/evaluation-tools/nucleo-f401re.html) |
| USB Cable | Connect board to PC | Type A to Mini-B |

{: .box-note}
**Good News!** The Nucleo board has a built-in **ST-LINK** that provides USB-to-UART conversion, so you don't need an external USB-to-Serial adapter!

---

##  Configure UART in STM32CubeMX

### Create New Project

1. Open **STM32CubeMX** and create a new project
2. Select your MCU: **STM32F401RETx**
3. Click **Start Project**

### Enable USART2

1. Navigate to **Connectivity → USART2**
2. Set Mode to **Asynchronous**
3. Keep the default parameters:
   - **Baud Rate**: 115200 Bits/s
   - **Word Length**: 8 Bits
   - **Parity**: None
   - **Stop Bits**: 1

{: .box-note}
**Why USART2?** On the Nucleo-F401RE, **USART2** is connected to the ST-LINK USB port (pins **PA2** and **PA3**), making it perfect for PC communication without extra wiring!

### Generate Code

1. Go to **Project Manager** tab
2. Set project name and location
3. Select **Toolchain/IDE**: STM32CubeIDE
4. Click **Generate Code**

---

##  Understanding UART Reception Methods

Before diving into code, let's understand the three ways to receive UART data:

| Method | How It Works | CPU Usage | Complexity | Best For |
|:-------|:-------------|:----------|:-----------|:---------|
| **Polling** | CPU constantly checks if data arrived | High (Blocking) | Simple | Small data, learning |
| **Interrupt** | Hardware alerts CPU when data arrives | Medium | Moderate | Real-time responses |
| **DMA** | Hardware transfers data directly to memory | Low | Complex | Large data streams |

### Detailed Comparison

{: .box-warning}
**Polling Method**  
 **Advantages**: Simple to implement, easy to debug, great for beginners  
 **Disadvantages**: CPU is blocked while waiting, wastes processing power, can't do other tasks

{: .box-note}
**Interrupt Method**  
 **Advantages**: Non-blocking, CPU free for other tasks, good responsiveness  
 **Disadvantages**: Requires interrupt service routines, moderate complexity, overhead per byte

{: .box-success}
**DMA Method**  
 **Advantages**: Fully non-blocking, minimal CPU load, ideal for high-speed data  
 **Disadvantages**: Most complex setup, requires DMA configuration, uses DMA channels

---

##  Write Code for UART Reception (Polling Mode)

Open your project in **STM32CubeIDE** and add this code to `main.c`:

```c
char msg[] = "STM32 UART Ready! Welcome\r\n";

while (1)
{
HAL_UART_Transmit(&huart2, (uint8_t *)msg, strlen(msg), HAL_MAX_DELAY); // Send welcome message
HAL_Delay(500);
}
```
text

###  Code Explanation

| Function | Purpose | Parameters |
|:---------|:--------|:-----------|
| `HAL_UART_Transmit()` | Send data to PC | handle, data, size, timeout |



**What's happening?**
1. We define a message `msg` to send to the terminal.
2. Inside the infinite loop, we call `HAL_UART_Transmit()` to send the message over USART2.
3. The function parameters specify the UART handle, data buffer, size of data, and timeout duration.


---

##  Setup PuTTY Terminal

### Find Your COM Port

1. Connect your Nucleo board via USB
2. Open **Device Manager** (Windows)
3. Expand **Ports (COM & LPT)**
4. Note the COM port (e.g., **COM3**) for "STMicroelectronics STLink Virtual COM Port"

### Configure PuTTY

1. Open **PuTTY**
2. Select **Connection type**: Serial
3. Enter **Serial line**: COM3 (your port number)
4. Set **Speed**: 115200
5. Go to **Connection → Serial**:
   - **Data bits**: 8
   - **Stop bits**: 1
   - **Parity**: None
   - **Flow control**: None
6. Click **Open**

{: .box-note}
**Pro Tip**: Save this configuration as a session named "STM32_UART" so you don't have to re-enter settings every time!

---

## Build and Run

1. Connect your STM32F401RE Nucleo board
2. Click **Build Project** (🔨 icon)
3. Click **Run** (▶️ icon) in STM32CubeIDE
4. Switch to your **PuTTY window**
5. You should see: `STM32 UART Ready! Welcome` contnuously printed.


---

##  Troubleshooting

{: .box-error}
**Problem**: Nothing appears in PuTTY!  
**Solutions**:
- Verify correct COM port in Device Manager
- Check baud rate is 115200 in both code and PuTTY
- Ensure USART2 is properly initialized
- Try unplugging and reconnecting the board

{: .box-error}
**Problem**: Garbage characters displayed!  
**Solutions**:
- Baud rate mismatch — verify both sides use 115200
- Check word length, parity, and stop bits match
- Ensure proper clock configuration



---

##  Reception Methods Summary

| Feature | Polling | Interrupt | DMA |
|:--------|:--------|:----------|:----|
| CPU Blocking |  Yes |  No |  No |
| Code Complexity |  Simple |  Moderate |  Complex |
| Power Efficiency |  Low |  Medium |  High |
| Data Rate | Low | Medium | High |
| Recommended For | Learning | Most projects | High-speed data |

---

##  Key Takeaways

| Concept | HAL Function |
|:--------|:-------------|
| Send data | `HAL_UART_Transmit()` |
| Receive data | `HAL_UART_Receive()` |
| Check status | Return value `== HAL_OK` |
| Timeout | Last parameter (milliseconds) |

---

##  Conclusion

{: .box-success}
Congratulations! You've successfully implemented **UART communication** between your STM32 Nucleo board and PC. You can now:
- Send and receive text via UART


> **Next Challenge**: Try implementing **interrupt-based reception** for non-blocking communication, or add command parsing to control LEDs via UART!

---

![STM32 UART Communication](https://vivekchadhuvula01.github.io/Lets-build/assets/img/uart.png){: .mx-auto.d-block :}

<details markdown="1">
<summary>Click here for bonus exercises</summary>

**Exercise 1**: Modify the code to turn an LED ON when you type "ON" and OFF when you type "OFF"

**Exercise 2**: Implement a simple calculator that receives two numbers and an operator (+, -, *, /) via UART and returns the result

**Exercise 3**: Create a menu system that displays options and responds to user input:
<!-- Toggle LED

Read sensor

Display status
Enter choice: -->

<!-- text -->

<!-- **Hint**: Use `strcmp()` to compare received strings with commands! -->
</details>