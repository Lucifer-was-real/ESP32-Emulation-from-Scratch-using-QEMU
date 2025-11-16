
ESP32 LED Blink on QEMU
# ESP32 LED Blink Project on QEMU  
#Link to the project files 
https://drive.google.com/drive/folders/1g5Zwg2nIl9OjoNWFba_46e6cQKdMfKQV?usp=drive_link

This document explains exactly how I set up ESP-IDF, built the LED Blink project, created the flash image, and successfully ran the firmware on the QEMU ESP32 emulator.  
It also includes the errors I faced and how I fixed them.

---

# 1. System Information
- OS: Ubuntu Linux (22.04)
- ESP-IDF Version: v5.1 
- QEMU Version: QEMU emulator version 9.2.2 (v9.0.0-5104-g18a7345ed4)
- Compiler: xtensa-esp32-elf-gcc  

---

# 2. Install ESP-IDF
---
```bash
bash
cd ~/esp
git clone -b v5.1 --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh
. ./export.sh
```
---
Verify:
```idf.py –version```
<img width="1834" height="98" alt="image" src="https://github.com/user-attachments/assets/d2bbdbbb-4ec3-4c52-a6d2-c000ecc5f506" />

Build Espressif’s QEMU for ESP32
```
cd ~
git clone https://github.com/espressif/qemu.git
cd qemu
git submodule update --init --recursive
./configure --target-list=xtensa-softmmu
make -j$(nproc)
```
Check executable:
```
ls build/qemu-system-xtensa
```
Later I discovered my working QEMU binaries were at:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa
/home/pranoymukhrjeee/qemu_old/build/qemu-system-xtensa
```
Create the LED Blink Project
```
cd ~/esp32_tasks
idf.py create-project blink_project
cd blink_project
idf.py set-target esp32
idf.py build
```
<img width="1852" height="347" alt="image" src="https://github.com/user-attachments/assets/6678a30e-12d5-4346-9ba6-bf2d53d8a389" />

This created:
```
build/bootloader/bootloader.bin
build/partition_table/partition-table.bin
build/blink.bin
```

Merge Binaries into one Flash Image
QEMU needs a single flash image, so I merged the binaries:
```
esptool.py --chip esp32 merge_bin \
  --flash_mode dio --flash_size 4MB -o build/flash_image.bin \
  0x1000 build/bootloader/bootloader.bin \
  0x8000 build/partition_table/partition-table.bin \
  0x10000 build/blink.bin
```
<img width="1059" height="166" alt="image" src="https://github.com/user-attachments/assets/eecb4a0e-2632-429f-b5ba-8695992e3c30" />

Verify:
```
ls -lh build/flash_image.bin
```
<img width="1066" height="46" alt="image" src="https://github.com/user-attachments/assets/d2a9978f-7304-4e8b-b86b-bd7185576f7d" />


Expected: ~4 MB
if not or greater than 4mb we can truncate the file using : 
```
truncate -s 4M build/flash_image.bin
```

Run Blink Firmware on QEMU
Working command:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa \
  -nographic \
  -machine esp32 \
  -drive file=build/flash_image.bin,if=mtd,format=raw
```
Output looked like:
Adding SPI flash device
ets Jul 29 2019 12:21:46

rst:0x1 (POWERON_RESET),boot:0x12 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:2
load:0x3fff0030,len:7076
load:0x40078000,len:15576
load:0x40080400,len:4
load:0x40080404,len:3876
entry 0x4008064c
I (737) boot: ESP-IDF v5.1 2nd stage bootloader
I (738) boot: compile time Nov 16 2025 11:07:10
I (738) boot: Multicore bootloader
I (767) boot: chip revision: v0.0
I (770) boot.esp32: SPI Speed      : 40MHz
I (770) boot.esp32: SPI Mode       : DIO
I (770) boot.esp32: SPI Flash Size : 4MB
I (782) boot: Enabling RNG early entropy source...
I (800) boot: Partition Table:
I (801) boot: ## Label            Usage          Type ST Offset   Length
I (802) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (804) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (805) boot:  2 factory          factory app      00 00 00010000 00100000
I (813) boot: End of partition table
I (823) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=09a98h ( 39576) map
I (862) esp_image: segment 1: paddr=00019ac0 vaddr=3ffb0000 size=02148h (  8520) load
I (898) esp_image: segment 2: paddr=0001bc10 vaddr=40080000 size=04408h ( 17416) load
I (944) esp_image: segment 3: paddr=00020020 vaddr=400d0020 size=149f0h ( 84464) map
I (981) esp_image: segment 4: paddr=00034a18 vaddr=40084408 size=07a34h ( 31284) load
I (1014) boot: Loaded app from partition at offset 0x10000
I (1014) boot: Disabling RNG early entropy source...
I (1029) cpu_start: Multicore app
I (1034) cpu_start: Pro cpu up.
I (1034) cpu_start: Starting app cpu, entry point is 0x400810d8
I (676) cpu_start: App cpu up.
I (1762) cpu_start: Pro cpu start user code
I (1763) cpu_start: cpu freq: 160000000 Hz
I (1763) cpu_start: Application information:
I (1763) cpu_start: Project name:     blink
I (1763) cpu_start: App version:      1
I (1763) cpu_start: Compile time:     Nov 16 2025 11:06:59
I (1764) cpu_start: ELF file SHA256:  69d7ca26a6aaaf6d...
I (1764) cpu_start: ELF file SHA256:  69d7ca26a6aaaf6d...
I (1764) cpu_start: ESP-IDF:          v5.1
I (1764) cpu_start: Min chip rev:     v0.0
I (1765) cpu_start: Max chip rev:     v3.99 
I (1765) cpu_start: Chip rev:         v0.0
I (1766) heap_init: Initializing. RAM available for dynamic allocation:
I (1767) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (1767) heap_init: At 3FFB29B0 len 0002D650 (181 KiB): DRAM
I (1768) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (1768) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (1768) heap_init: At 4008BE3C len 000141C4 (80 KiB): IRAM
I (1815) spi_flash: detected chip: gd
I (1821) spi_flash: flash io: dio
I (1838) app_start: Starting scheduler on CPU0
I (1841) app_start: Starting scheduler on CPU1
I (1841) main_task: Started on CPU0
I (1851) main_task: Calling app_main()
I (1851) example: Example configured to blink GPIO LED!
I (1851) gpio: GPIO[5]| InputEn: 0| OutputEn: 0| OpenDrain: 0| Pullup: 1| Pulldown: 0| Intr:0 
I (1851) example: Turning the LED OFF!
I (2851) example: Turning the LED ON!
I (3851) example: Turning the LED OFF!
I (4851) example: Turning the LED ON!
I (5851) example: Turning the LED OFF!
I (6851) example: Turning the LED ON!
I (7851) example: Turning the LED OFF!
<img width="1144" height="1005" alt="image" src="https://github.com/user-attachments/assets/86e10bfa-6830-4387-8146-cb6086253bd7" />

Problems I Faced & Fixes
Problem 1 — idf.py: command not found
Fix: ESP-IDF environment was not exported.
```
. ~/esp/esp-idf/export.sh
```

Problem 2 — Flash image not found
Could not open 'flash_image.bin'
Fix: I was running QEMU from the wrong directory.
Solution: run QEMU inside the project folder or give full path.

Problem 3 — esp32.flash_size has invalid class name
This happened because I used an outdated QEMU command.
Fix: remove -global and use simple flash drive syntax:
```
-drive file=build/flash_image.bin,if=mtd,format=raw
```

Problem 4 — QEMU not found (No such file or directory)
I had multiple QEMU builds, only two were valid.
Fix: located them with:
```
find ~ -name "qemu-system-xtensa"
```
Then used:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa
```

Final Working Command
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa \
  -nographic \
  -machine esp32 \
  -drive file=build/flash_image.bin,if=mtd,format=raw
```
This successfully ran the LED blink project.




The goal of FOSSEE's Yaksh platform is to provide immediate verification of code solutions for various languages (Python, C, C++, Java, etc.). 
My successful setup of the ESP32 project running within the QEMU emulator creates a powerful and isolated environment that can be leveraged for automatically grading embedded systems code on Yaksh.
I. Yaksh's Existing Code Evaluation Model
The Yaksh system already supports arbitrary coding questions (Page 3 of the Yaksh documentation). It achieves code verification using a sandboxed environment (Docker or local unsafe mode) and relies on background tasks (Celery/Redis) to re-evaluate submissions.
For standard languages like C or Python, the workflow is:
    1. Student submits code.
    2. Yaksh's Code Server compiles/executes the code in a sandbox.
    3. The sandbox returns the program's output (stdout/stderr) or an exit code.
    4. Yaksh compares this output against a pre-defined expected output or executes grading scripts.
II. Integrating ESP32/QEMU for Automated Grading
To use your current ESP32/QEMU setup within Yaksh, the evaluation logic needs to be modified to account for the unique, multi-step nature of embedded development: (Compile) -> (Flash Image) -> (Emulate/Run) -> (Check Log Output).



<b>ESP32 Temperature Sensor Simulation on QEMU</b>
<br>
--

# 1. System Information
- OS: Ubuntu Linux (22.04)
- ESP-IDF Version: v5.1 
- QEMU Version: QEMU emulator version 9.2.2 (v9.0.0-5104-g18a7345ed4)
- Compiler: xtensa-esp32-elf-gcc  

---
<br> WE CAN SKIP THE INSTALLATION PART SINCE ITS DONE ALREADY </br>
# 2. Install ESP-IDF
---
```bash
bash
cd ~/esp
git clone -b v5.1 --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh
. ./export.sh
```
---
Verify:
```idf.py –version```
<img width="1834" height="98" alt="image" src="https://github.com/user-attachments/assets/d2bbdbbb-4ec3-4c52-a6d2-c000ecc5f506" />

Build Espressif’s QEMU for ESP32
```
cd ~
git clone https://github.com/espressif/qemu.git
cd qemu
git submodule update --init --recursive
./configure --target-list=xtensa-softmmu
make -j$(nproc)
```
Check executable:
```
ls build/qemu-system-xtensa
```
Later I discovered my working QEMU binaries were at:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa
/home/pranoymukhrjeee/qemu_old/build/qemu-system-xtensa
```

Create the Temperature Project
Navigate to your workspace:
```
cd ~/esp32_tasks
```
Create project:
```
idf.py create-project temp_project
```
Enter project:
```
cd temp_project
```
Set target:
```
idf.py set-target esp32
```

4. Temperature Simulation Code
Replace main/temp_project.c with:
```
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_random.h"

float get_simulated_temperature() {
    return 20.0 + (esp_random() % 2000) / 100.0;
}

void app_main() {
    while (1) {
        float temp = get_simulated_temperature();
        printf("Simulated Temperature: %.2f °C\n", temp);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

Build the Firmware
Run:
```
idf.py build
```
This generates:
    • build/bootloader/bootloader.bin
    • build/partition_table/partition-table.bin
    • build/temp_project.bin
    • build/temp_project.elf
Check files:
```
ls build/*.bin
ls build/*.elf
```
<img width="888" height="75" alt="image" src="https://github.com/user-attachments/assets/827b6cff-7d5d-4aaf-b783-fa21c356b20e" />

Create Flash Image for QEMU
Merge binaries into a 4MB flash image:
```
esptool.py --chip esp32 merge_bin -o flash_image.bin \
  --flash_mode dio --flash_size 4MB \
  0x1000 build/bootloader/bootloader.bin \
  0x8000 build/partition_table/partition-table.bin \
  0x10000 build/temp_project.bin
```
Verify:
```
ls -lh flash_image.bin
```
<img width="951" height="42" alt="image" src="https://github.com/user-attachments/assets/31d1f8c5-75cb-4aac-986e-c9240bc1ca64" />


Locate Working QEMU Binary
Because multiple QEMU versions existed, we searched:
```
find ~ -name "qemu-system-xtensa"
```
Working binaries found:
```
<img width="1053" height="97" alt="image" src="https://github.com/user-attachments/assets/ca24fbf3-fde6-48ba-b152-983ca9a2f61e" />


~/qemu_backup/build/qemu-system-xtensa
~/qemu_old/build/qemu-system-xtensa
```
We used:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa
```

Run Temperature Project on QEMU
Run inside the project folder:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa \
  -nographic \
  -machine esp32 \
  -drive file=flash_image.bin,if=mtd,format=raw
```
Expected output:
<img width="1284" height="993" alt="image" src="https://github.com/user-attachments/assets/87f6b8f6-d2d5-4ff0-913b-410d589a8105" />

<img width="703" height="564" alt="image" src="https://github.com/user-attachments/assets/ac578436-0de7-4c27-bede-7ebe6c9d47a6" />


Problems Faced (Detailed)
idf.py: command not found
Cause: ESP-IDF environment was not activated.
Fix:
```
. ~/esp/esp-idf/export.sh
```

Build Failed — Missing esp_random()
Error:
implicit declaration of function 'esp_random'
Cause: ESP-IDF v5 removed automatic inclusion of esp_random.h.
Fix: Added:
```
#include "esp_random.h"
```

Missing temp_project.bin
Symptom:
```ls build/*.bin``` → no file found
Cause: Build failed before generating binaries due to compiler error.
Fix: After fixing esp_random() issue, reboot build:
```idf.py build```

Wrong QEMU Path (No such file or directory)
Attempted to run:
```~/qemu/build/qemu-system-xtensa```   ← INVALID PATH
Fix: Locate real QEMU binaries:
```
find ~ -name "qemu-system-xtensa"
```
Found correct paths:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa
/home/pranoymukhrjeee/qemu_old/build/qemu-system-xtensa
```
Used:
```
/home/pranoymukhrjeee/qemu_backup/build/qemu-system-xtensa
```

Incorrect QEMU Flash Device Arguments
Error:
-device esp32.flash is not a valid device model
Cause: Old documentation used outdated QEMU syntax.
Fix: Correct argument:
```
-drive file=flash_image.bin,if=mtd,format=raw
```

Flash image not found by QEMU
QEMU error:
Could not open 'flash_image.bin'
Cause: Running QEMU from a different directory.
Fix: Run QEMU inside the project folder or provide full path:
```
-drive file=/home/.../flash_image.bin,if=mtd,format=raw
```

Merge command failed (wrong .bin name)
Error:
No such file or directory: build/temp_project.bin
Cause: File name mismatch.
Fix: Verified actual binary name via:
```
ls build/*.bin
```


