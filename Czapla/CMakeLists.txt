#THIS FILE IS AUTO GENERATED FROM THE TEMPLATE! DO NOT CHANGE!
SET(CMAKE_SYSTEM_NAME Generic)
SET(CMAKE_SYSTEM_VERSION 1)
cmake_minimum_required(VERSION 3.11)


# specify cross compilers and tools
SET(CMAKE_C_COMPILER_WORKS 1)
SET(CMAKE_C_COMPILER arm-none-eabi-gcc)
SET(CMAKE_CXX_COMPILER_WORKS 1)
SET(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER  arm-none-eabi-gcc)
set(CMAKE_AR arm-none-eabi-ar)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP arm-none-eabi-objdump)
set(SIZE arm-none-eabi-size)

PROJECT(Czapla C CXX ASM)
set(CMAKE_CXX_STANDARD 11)

SET(LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/STM32F446RETx_FLASH.ld)

#Uncomment for hardware floating point
#SET(FPU_FLAGS "-mfloat-abi=hard -mfpu=fpv4-sp-d16")
#add_definitions(-DARM_MATH_CM4 -DARM_MATH_MATRIX_CHECK -DARM_MATH_ROUNDING -D__FPU_PRESENT=1)

#Uncomment for software floating point
#SET(FPU_FLAGS "-mfloat-abi=soft")
SET(COMMON_FLAGS
    "-mcpu=cortex-m4 ${FPU_FLAGS} -mthumb -mthumb-interwork -ffunction-sections -fdata-sections \
    -g -fno-common -fmessage-length=0 -specs=nosys.specs -specs=nano.specs")

SET(CMAKE_CXX_FLAGS_INIT "${COMMON_FLAGS} -std=c++11")
SET(CMAKE_C_FLAGS_INIT "${COMMON_FLAGS} -std=gnu99")
SET(CMAKE_EXE_LINKER_FLAGS_INIT "-Wl,-gc-sections,--print-memory-usage -T ${LINKER_SCRIPT}")


#add_definitions(-DARM_MATH_CM4 -DARM_MATH_MATRIX_CHECK -DARM_MATH_ROUNDING -D__FPU_PRESENT=1)
add_definitions(-D__weak=__attribute__\(\(weak\)\) -D__packed=__attribute__\(\(__packed__\)\) -DUSE_HAL_DRIVER -DSTM32F446xx)
add_compile_options(-Wall)
add_link_options("-u_printf_float")

if(${VERBOSE_TEST_OUTPUT})
    add_compile_definitions(VERBOSE_TEST_OUTPUT UNITY_OUTPUT_COLOR)
endif()
if(${CMAKE_BUILD_TYPE} STREQUAL Test)
    add_compile_definitions(RUN_TESTS)
endif()
if(${CMAKE_BUILD_TYPE} STREQUAL Simulate-test)
    add_compile_definitions(SIMULATE RUN_TESTS)
endif()
if(${CMAKE_BUILD_TYPE} STREQUAL Simulate)
    add_compile_definitions(SIMULATE)
endif()

file(GLOB_RECURSE SOURCES
                "Drivers/*.*"
                "Inc/*.*"
                "Src/*.*"
                "startup/*.*"
                "Middlewares/*.*")

include_directories(Inc
                    Test
                    Drivers/STM32F4xx_HAL_Driver/Inc
                    Drivers/STM32F4xx_HAL_Driver/Inc/Legacy
                    Drivers/CMSIS/Device/ST/STM32F4xx/Include
                    Drivers/CMSIS/Include
                    Middlewares/ST/STM32_USB_Device_Library/Core/Inc
                    Middlewares/ST/STM32_USB_Device_Library/Class/MSC/Inc)

add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LINKER_SCRIPT})

# DEFINE YOUR LIBRARIES HERE
add_compile_definitions(SS_USE_COM)
add_compile_definitions(SS_USE_GRAZYNA)
add_compile_definitions(SS_USE_FIFO)


add_subdirectory(../SS-common common)

target_link_libraries(${PROJECT_NAME}.elf common)

set(CMAKE_EXE_LINKER_FLAGS
    "${CMAKE_EXE_LINKER_FLAGS} -Wl,-Map=${PROJECT_BINARY_DIR}/${PROJECT_NAME}.map")

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)


add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
        COMMENT "Building ${HEX_FILE}
Building ${BIN_FILE}")

if(NOT DEFINED ENV{JUMPER})
    set(JUMPER ../../.venv/bin/jumper)
else()
    set(JUMPER $ENV{JUMPER})
endif()

if(${CMAKE_BUILD_TYPE} STREQUAL Simulate OR ${CMAKE_BUILD_TYPE} STREQUAL Simulate-test)
    add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
            COMMAND ${JUMPER} run --fw ${PROJECT_NAME}.bin -u UART5 --platform stm32f446)
endif()
