cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

set(PICO_BOARD pico CACHE STRING "Board type")

set(MS_RATE_DEFAULT 100 CACHE STRING "Default mouse sample rate")
set(MS_RATE_HOST_CONTROL ON CACHE BOOL "Allow the host to configure the mouse sample rate")

# Pull in Raspberry Pi Pico SDK
include(pico_sdk_import.cmake)

if (PICO_SDK_VERSION_STRING VERSION_LESS "1.5.1")
  message(FATAL_ERROR "Raspberry Pi Pico SDK version 1.5.1 (or later) required. Your version is ${PICO_SDK_VERSION_STRING}")
endif()

project(ps2x2pico C CXX ASM)

# Initialise the Raspberry Pi Pico SDK
pico_sdk_init()

add_executable(ps2x2pico src/ps2x2pico.c src/scancodesets.c src/ps2kb.c src/ps2ms.c src/ps2out.c src/ps2in.c)

pico_generate_pio_header(ps2x2pico ${CMAKE_CURRENT_LIST_DIR}/src/ps2out.pio)
pico_generate_pio_header(ps2x2pico ${CMAKE_CURRENT_LIST_DIR}/src/ps2in.pio)

### PS/2 output
add_compile_definitions(LVOUT=5) # Level shifter pull-up voltage
add_compile_definitions(KBOUT=2) # Keyboard data pin, clock pin is 3
add_compile_definitions(MSOUT=6) # Mouse data pin, clock pin is 7

### PS/2 input
add_compile_definitions(LVIN=13) # Level shifter pull-up voltage
add_compile_definitions(KBIN=11) # Keyboard data pin, clock pin is 12
add_compile_definitions(MSIN=14) # Mouse data pin, clock pin is 15

add_compile_definitions(PICO_PANIC_FUNCTION=reset)

add_compile_definitions(MS_RATE_DEFAULT=${MS_RATE_DEFAULT})
if (MS_RATE_HOST_CONTROL)
    add_compile_definitions(MS_RATE_HOST_CONTROL)
endif()

pico_set_program_name(ps2x2pico "ps2x2pico")
pico_set_program_version(ps2x2pico "1.3")

pico_enable_stdio_uart(ps2x2pico 1)
pico_enable_stdio_usb(ps2x2pico 0)

target_include_directories(ps2x2pico PRIVATE ${CMAKE_CURRENT_LIST_DIR}/src)
target_link_libraries(ps2x2pico pico_stdlib hardware_pio tinyusb_host tinyusb_board)

pico_add_extra_outputs(ps2x2pico)
