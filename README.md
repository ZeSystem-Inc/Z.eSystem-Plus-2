# ZeSystemPlus 🚀 (v2.0.0 - Storage, Filesystem & Hardware Discovery)

Welcome to the **v2.0.0** release of **ZeSystemPlus**! Building upon our initial protected-mode core baseline, this major architectural update transforms the kernel into a functional, bare-metal ecosystem featuring a custom filesystem, dynamic hardware scanning, low-level storage drivers, and a native system installer.

## 🛠️ Advanced Subsystems & Features

This release integrates crucial operating system subsystems, expanding the kernel's capability to interact directly with hardware components:

1. **Custom Filesystem (ESFS - `esfs.cpp`)**
   - Implements our proprietary **Easy/Simple File System**.
   - Custom-tailored from scratch to handle file structures, indexing, and efficient storage layouts without relying on standard commercial abstractions.

2. **PCI Bus Enumeration (`pci.cpp`)**
   - Introduces advanced hardware discovery capabilities.
   - Dynamically scans and enumerates the PCI bus architecture to identify connected hardware components, controllers, and peripherals.

3. **IDE Disk Driver & Text Scrolling (`disk.cpp`)**
   - Integrated low-level IDE/ATA disk controller driver allowing sector-level read/write operations to persistent storage media.
   - Incorporates an enhanced terminal screen scrolling implementation inside the VGA buffer to handle system logs and runtime output smoothly.

4. **Dedicated System Installer (`installer.cpp`)**
   - A built-in deployment module capable of handling system installation.
   - Safely commits the boot sectors and raw kernel binary strings directly onto an active storage drive via the newly implemented IDE driver.

## 🚀 Compilation & Build System

With the addition of multiple modular subsystems, the compilation toolchain now compiles separate object files and links them tightly together:

```bash
rm -f *.o *.bin os.img hdd.img

nasm -f bin boot.asm -o boot.bin

g++ -m32 -ffreestanding -fno-pie -fno-rtti -fno-exceptions -c kernel.cpp -o kernel.o
g++ -m32 -ffreestanding -fno-pie -fno-rtti -fno-exceptions -c disk.cpp -o disk.o
g++ -m32 -ffreestanding -fno-pie -fno-rtti -fno-exceptions -c esfs.cpp -o esfs.o
g++ -m32 -ffreestanding -fno-pie -fno-rtti -fno-exceptions -c installer.cpp -o installer.o
g++ -m32 -ffreestanding -fno-pie -fno-rtti -fno-exceptions -c pci.cpp -o pci.o

ld -m elf_i386 -T linker.ld --oformat binary kernel.o disk.o esfs.o installer.o pci.o -o kernel.bin

truncate -s 20480 kernel.bin
cat boot.bin kernel.bin > os.img
truncate -s 10M os.img

qemu-img create -f raw hdd.img 10M

qemu-system-i386 -drive file=hdd.img,format=raw,if=none,id=drive-hdd -device ide-hd,bus=ide.0,unit=0,drive=drive-hdd -drive file=os.img,format=raw,if=none,id=drive-os -device ide-hd,bus=ide.0,unit=1,drive=drive-os,bootindex=1

qemu-system-i386 -drive file=hdd.img,format=raw,index=0,media=disk
