cmake_minimum_required(VERSION 3.15.3)

project(nu-cmake-example)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

SET(M480BSP $ENV{M480BSP})

set(CMSIS_SRC
        ${M480BSP}/Library/Device/Nuvoton/M480/Source/system_M480.c
        ${M480BSP}/Library/Device/Nuvoton/M480/Source/GCC/startup_M480.S
        ${M480BSP}/Library/Device/Nuvoton/M480/Source/GCC/_syscalls.c
        )

set(DRV_SRC
        #${M480BSP}/Library/StdDriver/src/retarget.c
        ${M480BSP}/Library/StdDriver/src/clk.c
        ${M480BSP}/Library/StdDriver/src/sys.c
        ${M480BSP}/Library/StdDriver/src/uart.c
        ${M480BSP}/Library/StdDriver/src/gpio.c
        )

set(EXECUTABLE ${PROJECT_NAME}.elf)

add_executable(${EXECUTABLE} ../User/main.c ${CMSIS_SRC} ${DRV_SRC})

#target_compile_definitions(${EXECUTABLE} PRIVATE)

target_include_directories(${EXECUTABLE} PRIVATE
        ${M480BSP}/Library/Device/Nuvoton/M480/Include
        ${M480BSP}/Library/StdDriver/inc
        ${M480BSP}/Library/CMSIS/Include
        )

target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=cortex-m4
        -mthumb
        -mfpu=fpv4-sp-d16
        -mfloat-abi=hard
        -fdata-sections
        -ffunction-sections
        -Os
        -std=gnu11
        -Wall
        $<$<CONFIG:Debug>:-Og>
        -fmessage-length=0 -fsigned-char
        -std=gnu11
        )

target_link_options(${EXECUTABLE} PRIVATE
        -T${M480BSP}/Library/Device/Nuvoton/M480/Source/GCC/gcc_arm.ld
        -mcpu=cortex-m4
        -mthumb
        -mfloat-abi=hard
        -mfpu=fpv4-sp-d16
        -Os
        -fmessage-length=0 -fsigned-char
        -ffunction-sections -fdata-sections
        -Xlinker 
        --gc-sections
        -Wl,-Map,${PROJECT_NAME}.map
        )

# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-size ${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-objcopy -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND arm-none-eabi-objcopy -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)

# Write to flash
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMENT "write to flash ..."
        COMMAND openocd -f ../flash.cfg
        )
