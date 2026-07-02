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
# 1. Assemble the 16-bit boot sector
nasm -f bin boot.asm -o boot.bin

# 2. Compile the core kernel and all advanced subsystems
i686-elf-g++ -c kernel.cpp -o kernel.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti
i686-elf-g++ -c esfs.cpp -o esfs.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti
i686-elf-g++ -c pci.cpp -o pci.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti
i686-elf-g++ -c disk.cpp -o disk.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti
i686-elf-g++ -c installer.cpp -o installer.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti

# 3. Link all compiled objects using the layout script
i686-elf-ld -T linker.ld -o kernel.bin kernel.o esfs.o pci.o disk.o installer.o --oformat binary

# 4. Generate the final bootable raw storage image
cat boot.bin kernel.bin > os.img
