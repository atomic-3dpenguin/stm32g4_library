cmake_minimum_required(VERSION 3.28)

project(STM32G4_FreeRTOS_Lib
VERSION 1.0.0
DESCRIPTION "Interface library for STM32G4 with HAL, FreeRTOS, and CMSIS."
HOMEPAGE_URL "https://github.com/atomic-3dpenguin/stm32g4_library"
LANGUAGES C CXX ASM)

# Set toolchain
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_SOURCE_DIR}/../../stm32-toolchain/arm-toolchain.cmake")
include(${CMAKE_TOOLCHAIN_FILE})

#Set some cmake variables
set(STM32G4_OBJ_LIB STM32G4_FreeRTOS_ObjLib)
set(STM32G4_INTERFACE_LIB STM32G4_FreeRTOS_Lib)

# ----------------------------------------------------------------------------
# Compiler and linker flags
# ----------------------------------------------------------------------------
set(LINKER_SCRIPT             "STM32G473QETX_FLASH.ld")
set(MCPU                    "-mcpu=Cortex-M4")
set(MFPU                    "-mfpu=fpv4-sp-d16")
set(MFLOAT_ABI              "-mfloat-abi=hard")
set(RUNTIME_LIBRARY         "--specs=nano.specs")
set(RUNTIME_LIBRARY_SYSCALLS "--specs=nosys.specs")

set(CMAKE_C_FLAGS "${MCPU} -std=gnu11 ${MFPU} ${MFLOAT_ABI} ${RUNTIME_LIBRARY} -mthumb -Wall -Werror")
set(CMAKE_ASM_FLAGS "${CMAKE_C_FLAGS} -x assembler-with-cpp")
set(CMAKE_EXE_LINKER_FLAGS "-T${LINKER_SCRIPT} ${RUNTIME_LIBRARY_SYSCALLS} -Wl,-Map=test.map -Wl,--gc-sections -static -Wl,--start-group -lc -lm -Wl,--end-group")

# ----------------------------------------------------------------------------
# Source files
# ----------------------------------------------------------------------------
file(GLOB_RECURSE HAL_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/STM32G4xx_HAL_Driver/Src/*.c
)

file(GLOB_RECURSE FREERTOS_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS/Source/*.c
)

file(GLOB_RECURSE STARTUP
    ${CMAKE_CURRENT_SOURCE_DIR}/Core/startup/*.s
)

# ----------------------------------------------------------------------------
# Object library
# ----------------------------------------------------------------------------
add_library(${STM32G4_OBJ_LIB} OBJECT
    ${HAL_SOURCES}
    ${FREERTOS_SOURCES}
    ${STARTUP}
)

target_include_directories(${STM32G4_OBJ_LIB} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Core/Inc>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Device/ST/STM32G4xx/Include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Drivers/STM32G4xx_HAL_Driver/Inc>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS/Source/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Middlewares/Third_Party/FreeRTOS/Source/portable/MemMnag>

    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Core/Inc>
    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Drivers/CMSIS/Include>
    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Drivers/CMSIS/Device/ST/STM32G4xx/Include>
    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Drivers/STM32G4xx_HAL_Driver/Inc>
    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Middlewares/Third_Party/FreeRTOS/Source/include>
    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS_V2>
    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Middlewares/Third_Party/FreeRTOS/Source/portable/GCC/ARM_CM4F>
    $<INSTALL_INTERFACE:include/${STM32G4_INTERFACE_LIB}/Middlewares/Third_Party/FreeRTOS/Source/portable/MemMnag>
)

target_compile_definitions(${STM32G4_OBJ_LIB} PUBLIC
    USE_HAL_DRIVER
    STM32G473xx
)

# ----------------------------------------------------------------------------
# Interface wrapper for clean linking
# ----------------------------------------------------------------------------
add_library(${STM32G4_INTERFACE_LIB} INTERFACE)
target_link_libraries(${STM32G4_INTERFACE_LIB} INTERFACE ${STM32G4_OBJ_LIB})

# ----------------------------------------------------------------------------
# Installation
# ----------------------------------------------------------------------------
include(GNUInstallDirs)

install(TARGETS
    ${STM32G4_OBJ_LIB}
    ${STM32G4_INTERFACE_LIB}
    EXPORT ${STM32G4_INTERFACE_LIB}Targets
)

install(EXPORT ${STM32G4_INTERFACE_LIB}Targets
    FILE ${STM32G4_INTERFACE_LIB}Targets.cmake
    NAMESPACE STM32::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${STM32G4_INTERFACE_LIB}
)

# Install headers
install(DIRECTORY Core/Inc/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${STM32G4_INTERFACE_LIB}/Core/Inc
    FILES_MATCHING PATTERN "*.h"
)
install(DIRECTORY Drivers/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${STM32G4_INTERFACE_LIB}/Drivers
    FILES_MATCHING PATTERN "*.h"
)
install(DIRECTORY Middlewares/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${STM32G4_INTERFACE_LIB}/Middlewares
    FILES_MATCHING PATTERN "*.h"
)

# ----------------------------------------------------------------------------
# CMake Package Config
# ----------------------------------------------------------------------------
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${STM32G4_INTERFACE_LIB}ConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    ${CMAKE_CURRENT_LIST_DIR}/cmake/TemplateConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/${STM32G4_INTERFACE_LIB}Config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${STM32G4_INTERFACE_LIB}
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${STM32G4_INTERFACE_LIB}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${STM32G4_INTERFACE_LIB}ConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${STM32G4_INTERFACE_LIB}
)

# ----------------------------------------------------------------------------
# pkg-config support
# ----------------------------------------------------------------------------
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/Template.pc.in
    ${CMAKE_CURRENT_BINARY_DIR}/${STM32G4_INTERFACE_LIB}.pc
    @ONLY
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${STM32G4_INTERFACE_LIB}.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)
