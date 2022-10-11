cmake_minimum_required(VERSION 3.17)
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(CMAKE_C_COMPILER armcc)
# set(CMAKE_CXX_COMPILER armcc)
# set(CMAKE_ASM_COMPILER armasm)
set(CMAKE_OBJCOPY fromelf)
set(CMAKE_TRY_COMPILE_TARGET_TYPE "STATIC_LIBRARY")

# project settings
project(template C ASM)
set(CMAKE_CXX_STANDARD 03)
set(CMAKE_C_STANDARD 99)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "ARMCC")
    set(ARMCC 1)
endif()

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "ARMClang")
    set(ARMClang 1)
endif()

if (ARMCC)
    set(c_flags --c99 --cpu Cortex-M4.fp.sp --apcs=interwork --split_sections)
    set(asm_flags -c --cpu Cortex-M4.fp.sp --apcs=interwork)
    add_compile_options(
        "$<$<COMPILE_LANGUAGE:C>:${c_flags}>"
        "$<$<COMPILE_LANGUAGE:ASM>:${asm_flags}>"
    )
endif()

add_compile_definitions(USE_HAL_DRIVER STM32G431xx)

include_directories(
        ../Core/Inc
        ../Drivers/STM32G4xx_HAL_Driver/Inc
        ../Drivers/STM32G4xx_HAL_Driver/Inc/Legacy
        ../Drivers/CMSIS/Device/ST/STM32G4xx/Include
        ../Drivers/CMSIS/Include
        )


file(GLOB SOURCES
    ../Core/Src/*.c
    ../Core/Src/*.s
    ../Drivers/STM32G4xx_HAL_Driver/Src/*.c
    ../Drivers/CMSIS/**/*.c
        )

set(LINKER_SCRIPT uart.sct)
set_target_properties(${TARGET_NAME} PROPERTIES LINK_DEPENDS ${LINKER_SCRIPT})
#add_link_options("--scatter=${LINKER_SCRIPT}")

add_executable(${PROJECT_NAME} ${SOURCES})

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(ASM_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.asm)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} --i32 $<TARGET_FILE:${PROJECT_NAME}> --output=${HEX_FILE}
        COMMAND ${CMAKE_OBJCOPY} --text -c --cpu=Cortex-M0 $<TARGET_FILE:${PROJECT_NAME}> --output=${ASM_FILE}
        COMMAND ${CMAKE_OBJCOPY} --bin $<TARGET_FILE:${PROJECT_NAME}> --output=${BIN_FILE}
        COMMENT "Building ${HEX_FILE}
Building ${ASM_FILE}
Building ${BIN_FILE}")
