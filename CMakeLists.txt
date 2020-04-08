cmake_minimum_required(VERSION 3.6)
#set(CMAKE_VERBOSE_MAKEFILE ON)

# system stuff
enable_language(ASM)
set(CMAKE_SYSTEM_NAME STM32)
set(CFLAGS "-mcpu=cortex-m3 -mthumb -Og -Wall -fdata-sections -ffunction-sections")
set(CMAKE_ASM_FLAGS "${CFLAGS}")
if(CMAKE_BUILD_TYPE MATCHES Debug)
    set(CFLAGS "${CFLAGS} -gdwarf-2")
endif()
set(CMAKE_CXX_FLAGS "${CFLAGS} -std=c++17 -fno-rtti") # -fno-exceptions
set(CMAKE_C_FLAGS "${CFLAGS} -std=gnu99")
set(CMAKE_EXE_LINKER_FLAGS "-specs=nano.specs -specs=nosys.specs -Wl,--gc-sections")
set(CMAKE_C_LINK_FLAGS "")
set(CMAKE_CXX_LINK_FLAGS "")
set(CMAKE_OSX_SYSROOT "")



# project
project(PROJECT_NAME)
set(MCU_FAMILY STM32F1xx)
set(MCU_LINE STM32F103xB)



# CMSIS
include_directories(
    Drivers/CMSIS/Device/ST/${MCU_FAMILY}/Include
    Drivers/CMSIS/Include
)
add_definitions(-D${MCU_LINE})



# HAL
set(HAL_PREFIX Drivers/${MCU_FAMILY}_HAL_DRIVER/)
set(HAL_SRC ${HAL_PREFIX}Src/${MCU_FAMILY}_hal)
add_definitions(-DUSE_HAL_DRIVER)

include_directories(
    ${HAL_PREFIX}Inc
    ${HAL_PREFIX}Inc/Legacy
)

set(HAL_SOURCES
    ${HAL_SRC}_tim.c
    ${HAL_SRC}_tim_ex.c
    ${HAL_SRC}.c
    ${HAL_SRC}_rcc.c
    ${HAL_SRC}_rcc_ex.c
    ${HAL_SRC}_gpio.c
    ${HAL_SRC}_gpio_ex.c
    ${HAL_SRC}_dma.c
    ${HAL_SRC}_cortex.c
    ${HAL_SRC}_pwr.c
    ${HAL_SRC}_flash.c
    ${HAL_SRC}_flash_ex.c
)



# RTOS
set(RTOS_PREFIX Middlewares/Third_Party/FreeRTOS/Source/)

include_directories(
    ${RTOS_PREFIX}include
    ${RTOS_PREFIX}CMSIS_RTOS
    ${RTOS_PREFIX}portable/GCC/ARM_CM3
)

file(GLOB RTOS_SOURCES
    ${RTOS_PREFIX}*.c
    ${RTOS_PREFIX}CMSIS_RTOS/*.c
    ${RTOS_PREFIX}portable/MemMang/heap_4.c
    ${RTOS_PREFIX}portable/GCC/ARM_CM3/*.c
)



# sources
include_directories(Inc)

file(GLOB_RECURSE USER_SOURCES
    Src/*.c
    Src/*.cpp
)



# targets
file(GLOB MCU_LINKER_SCRIPT STM32*_FLASH.ld)
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -T ${MCU_LINKER_SCRIPT} -Wl,-Map=${PROJECT_NAME}.map,--cref")

add_executable(${PROJECT_NAME}.elf
    ${USER_SOURCES}
    ${HAL_SOURCES}
    ${RTOS_SOURCES}
    startup_${MCU_LINE}.s
)

target_link_libraries(${PROJECT_NAME}.elf
    c
    m
    nosys
)

add_custom_target(${PROJECT_NAME}.bin ALL DEPENDS ${PROJECT_NAME}.elf
    COMMAND ${CMAKE_OBJCOPY} -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
)
