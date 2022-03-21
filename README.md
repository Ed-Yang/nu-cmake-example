# Debug Nuvoton Cortex-M with VSCode/CMake and OpenOCD

This project demos :

1) Use gcc-arm-none-eabi/CMake to build Nuvoton Cortex-M program,
2) Utilize VSCode/Cortex-debug/OpenOCD to perform debugging.

Note, this mechanism is only been tested on "THIS" simple test program, use it on your risk.

```shell
+------------------------+
+ VSCode w/              +                    +------------------+
+ Cortex-Debug Extension + <--- USB (SWD) --->+ Nuvoton-PFM-M487 +    
+                        +                    +------------------+
+-------------------------
```

## Source Tree

```shell
./nu-cmake-example  
├── cmake  
|   └── arm-none-eabi-gcc.cmake  
├── CMakeLists.txt  
├── flash.cfg  
├── README.md  
├── res  
│   ├── M480.svd  <-- M480BSP/Library/CMSIS/SVD/ARM_Example.svd  
│   ├── nulink.cfg <-- OpenOCD-Nuvoton/tcl/interface/nulink.cfg  
│   ├── numicroM0.cfg <-- OpenOCD-Nuvoton/tcl/target/numicroM0.cfg  
│   └── numicroM4.cfg <-- OpenOCD-Nuvoton/tcl/target/numicroM4.cfg  
└── User  
    └── main.c  
```

## Environment

    ```shell
    export PROJ_ROOT=`pwd`     
    export M480BSP="folder-of-cloned-Nuvoton-M480BSP"  
    ```

## Build and Install OpenOCD

### Linux

* Install packages:

    sudo apt install gcc make automake autoconf pkg-config libtool libusb-1.0-0-dev libftdi-dev texinfo   
    sudo apt install cmake gcc-arm-none-eabi gdb-multiarch  

* Build and Install Nuvoton-OpenOCD:

    git clone https://github.com/OpenNuvoton/OpenOCD-Nuvoton  
    cd OpenOCD-Nuvoton  
    ./bootstrap  
    ./configure --disable-werror --disable-shared --enable-ftdi  
    make  
    make install  

    sudo cp ./contrib/60-openocd-nuvoton.rules /etc/udev/rules.d/  
    reboot  

* Test OpenOCD setup:

    Connect host USB to NuMaker-PFM-M487 and run:

    cd $PROJ_ROOT  
    openocd -f ./res/nulink.cfg -f ./res/numicroM4.cfg  

    ```shell
    Open On-Chip Debugger 0.10.0-dev-00477-g57b40002 (2022-03-20-08:29)
    Licensed under GNU GPL v2
    For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
    Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
    Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
    Info : clock speed 1000 kHz
    Info : NULINK is Nu-Link1
    Info : NULINK firmware_version(7313), product_id(0x40012009)
    Info : IDCODE: 0x2BA01477
    Info : NuMicro.cpu: hardware has 6 breakpoints, 4 watchpoints
    ```

### Windows

TBD

## Build Example Program

* Clone BSP source tree:

    git clone https://github.com/OpenNuvoton/M480BSP  
    export M480BSP=`pwd`/M480BSP  

* Build and Write to Flash

    cd $PROJ_ROOT  
    mkdir -p build && cd build  
    cmake -DCMAKE_TOOLCHAIN_FILE=../cmake/arm-none-eabi-gcc.cmake -DCMAKE_BUILD_TYPE=Debug ..  
    make  

    ```shell
    Scanning dependencies of target nu-cmake-example.elf
    [ 11%] Building C object CMakeFiles/nu-cmake-example.elf.dir/User/main.c.obj
    [ 22%] Linking C executable nu-cmake-example.elf
    text    data     bss     dec     hex filename
    30704    2536      92   33332    8234 nu-cmake-example.elf
    write to flash ...
    Open On-Chip Debugger 0.10.0-dev-00477-g57b40002 (2022-03-20-08:29)
    Licensed under GNU GPL v2
    For bug reports, read
            http://openocd.org/doc/doxygen/bugs.html
    Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
    Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
    Info : clock speed 1000 kHz
    Info : NULINK is Nu-Link1
    Info : NULINK firmware_version(7313), product_id(0x40012009)
    Info : IDCODE: 0x2BA01477
    Info : NuMicro.cpu: hardware has 6 breakpoints, 4 watchpoints
    auto erase enabled
    Info : Device ID: 0x00d48750
    Info : Device Name: M487JIDAE
    Info : bank base = 0x00000000, size = 0x10080000
    Info : Nuvoton NuMicro: Sector Erase ... (0 to 8)
    Info : Nuvoton NuMicro: Flash Write ...
    Info : Have written 19%
    Info : Have written 39%
    Info : Have written 59%
    Info : Have written 79%
    Info : Have written 98%
    Info : Have written 100%
    wrote 36864 bytes from file nu-cmake-example.hex in 16.331968s (2.204 KiB/s)
    shutdown command invoked
    [100%] Built target nu-cmake-example.elf
    ```

* Run Debugger

    Click menu "Run" --> "Start Debuggging", the debugger will break at "main" function.

## Test with GDB

Open one terminal to run OpenOCD:

    cd $PROJ_ROOT  
    openocd -f ./res/nulink.cfg -f ./res/numicroM4.cfg  

Open another terminal to run GDB:

    cd $PROJ_ROOT  
    gdb-multiarch ./build/nu-cmake-example.elf  

    In GDB session, issue the following commands, it will break on main function:

    ```shell
    target remote localhost:3333
    set remotetimeout 20
    set breakpoint pending on
    monitor reset init
    monitor halt
    monitor flash write_image erase ./build/nu-cmake-example.hex
    monitor reset init
    tbreak main
    continue
    list
    ```

## Reference

* [VSCode Cortex-Debug Extension Wiki](https://github.com/Marus/cortex-debug/wiki)

* [CMake separate flags for Assembler](https://stackoverflow.com/questions/65371607/cmake-separate-flags-for-assembler)

gcc -c foo.S will run your asm through the C preprocessor (before feeding it to as) if the filename has a .S (capital S) extension, not for lower-case .s. If you want to override that, yes -x <language> can be used, in this case -x assembler-with-cpp. (Not -x assembler, that's already the default for .s files). Use gcc -v foo.S to see what it runs.

