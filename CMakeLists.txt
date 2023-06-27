cmake_minimum_required(VERSION 3.25)
project(NDSGame)

set(CMAKE_CXX_STANDARD 23)

#---------------------------------------------------------------------------------
# Compiler options
#---------------------------------------------------------------------------------
set(DEVKITARM $ENV{DEVKITARM}) # Set DEVKITARM environment variable

# Check if DEVKITARM is set
if(NOT DEVKITARM)
    message(FATAL_ERROR "Please set DEVKITARM in your environment. export DEVKITARM=<path to>devkitARM")
endif()

set(CMAKE_C_COMPILER ${DEVKITARM}/bin/arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER ${DEVKITARM}/bin/arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER ${DEVKITARM}/bin/arm-none-eabi-as)

set(CMAKE_C_FLAGS "-g -Wall -O2 -march=armv5te -mtune=arm946e-s -fomit-frame-pointer -ffast-math ${ARCH} -DARM9")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -fno-rtti -fno-exceptions")
set(CMAKE_ASM_FLAGS "-g ${ARCH}")

#---------------------------------------------------------------------------------
# Source files and directories
#---------------------------------------------------------------------------------
set(TARGET ${PROJECT_NAME})
set(SOURCES gfx source data)
set(INCLUDES include build)

# List of directories containing libraries
set(LIBDIRS ${LIBNDS})

#---------------------------------------------------------------------------------
# Options for code generation
#---------------------------------------------------------------------------------
set(ARCH "-mthumb -mthumb-interwork")

#---------------------------------------------------------------------------------
# Any extra libraries we wish to link with the project
#---------------------------------------------------------------------------------
set(LIBS -lnds9)

#---------------------------------------------------------------------------------
# Set output paths
#---------------------------------------------------------------------------------
set(OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/${TARGET})
set(VPATH ${SOURCES})
set(DEPSDIR ${CMAKE_BINARY_DIR}/build)

#---------------------------------------------------------------------------------
# List of source files
#---------------------------------------------------------------------------------
file(GLOB CFILES ${CMAKE_CURRENT_SOURCE_DIR}/gfx/*.c ${CMAKE_CURRENT_SOURCE_DIR}/source/*.c ${CMAKE_CURRENT_SOURCE_DIR}/data/*.c)
file(GLOB CPPFILES ${CMAKE_CURRENT_SOURCE_DIR}/gfx/*.cpp ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp ${CMAKE_CURRENT_SOURCE_DIR}/data/*.cpp)
file(GLOB SFILES ${CMAKE_CURRENT_SOURCE_DIR}/gfx/*.s ${CMAKE_CURRENT_SOURCE_DIR}/source/*.s ${CMAKE_CURRENT_SOURCE_DIR}/data/*.s)
file(GLOB BINFILES ${CMAKE_CURRENT_SOURCE_DIR}/gfx/*.bin ${CMAKE_CURRENT_SOURCE_DIR}/source/*.bin ${CMAKE_CURRENT_SOURCE_DIR}/data/*.bin)

#---------------------------------------------------------------------------------
# Use CXX for linking C++ projects, CC for standard C
#---------------------------------------------------------------------------------
if(CPPFILES)
    set(LD ${CMAKE_CXX_COMPILER})
else()
    set(LD ${CMAKE_C_COMPILER})
endif()

#---------------------------------------------------------------------------------
# Set include directories
#---------------------------------------------------------------------------------
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/${INCLUDES})
foreach(dir ${LIBDIRS})
    include_directories(${dir}/include)
endforeach()

#---------------------------------------------------------------------------------
# Set library paths
#---------------------------------------------------------------------------------
foreach(dir ${LIBDIRS})
    link_directories(${dir}/lib)
endforeach()

#---------------------------------------------------------------------------------
# Build executable
#---------------------------------------------------------------------------------
add_executable(${OUTPUT}.elf ${BINFILES} ${CPPFILES} ${CFILES} ${SFILES})
target_link_libraries(${OUTPUT}.elf ${LIBS})

#---------------------------------------------------------------------------------
# Output files
#---------------------------------------------------------------------------------
add_custom_target(
        nds_output ALL
        DEPENDS ${OUTPUT}.nds ${OUTPUT}.ds.gba
)

add_custom_command(
        OUTPUT ${OUTPUT}.nds
        COMMAND ${DEVKITARM}/bin/ndstool -c ${OUTPUT}.nds -9 ${OUTPUT}.elf
        DEPENDS ${OUTPUT}.elf
)

add_custom_command(
        OUTPUT ${OUTPUT}.ds.gba
        COMMAND ${DEVKITARM}/bin/ndstool -c ${OUTPUT}.ds.gba -7 ${OUTPUT}.elf
        DEPENDS ${OUTPUT}.elf
)

#---------------------------------------------------------------------------------
# Clean target
#---------------------------------------------------------------------------------
add_custom_target(
        clean
        COMMAND ${CMAKE_COMMAND} -E remove_directory ${DEPSDIR}
        COMMAND ${CMAKE_COMMAND} -E remove ${OUTPUT}.elf ${OUTPUT}.nds ${OUTPUT}.ds.gba
)
