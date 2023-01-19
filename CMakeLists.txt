#
# Copyright 2019, Data61, CSIRO (ABN 41 687 119 230)
#
# SPDX-License-Identifier: BSD-2-Clause
#

cmake_minimum_required(VERSION 3.7.2)

include(settings.cmake)

project(sel4test C CXX ASM)

find_package(seL4 REQUIRED)
find_package(elfloader-tool REQUIRED)

# Set the roottask CNode size to be large enough for DTB, timer caps, etc
# Note that this works for the platforms that we have at the moment, and may
# need to be increased in the future
set(KernelRootCNodeSizeBits 13 CACHE INTERNAL "")

# Set our custom domain schedule
# set(KernelDomainSchedule "${CMAKE_CURRENT_LIST_DIR}/domain_schedule.c" CACHE INTERNAL "")
sel4_import_kernel()

include(ExternalProject)

foreach(overlay RANGE 1 4)
    ExternalProject_Add(seL4-${overlay}
        SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/kernel
        BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/seL4-${overlay}
        BUILD_ALWAYS ON
        BUILD_COMMAND cmake . && ninja kernel.elf
        INSTALL_COMMAND true
        
        CMAKE_ARGS
        "-DKernelIsMCS=${KernelIsMCS}"
        "-DPLATFORM=${PLATFORM}"
        "-DRELEASE=${RELEASE}"
        "-DVERIFICATION=${VERIFICATION}"
        "-DMCS=${MCS}"
        "-DKernelSel4Arch=${KernelSel4Arch}"
        "-DOVERLAY=${CMAKE_CURRENT_SOURCE_DIR}/overlay/overlay${overlay}.dts"
    )
    add_executable(seL4-${overlay}-kernel IMPORTED GLOBAL)
    set_property(
    TARGET seL4-${overlay}-kernel
    PROPERTY IMPORTED_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/seL4-${overlay}/kernel/kernel.elf"
    )
    add_dependencies(seL4-${overlay}-kernel seL4-${overlay})

    add_executable(seL4-${overlay}-dtb IMPORTED GLOBAL)
    set_property(
    TARGET seL4-${overlay}-dtb
    PROPERTY IMPORTED_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/seL4-${overlay}/kernel/kernel.dtb"
    )
    add_dependencies(seL4-${overlay}-dtb seL4-${overlay})


    add_executable(rootserver-${overlay} IMPORTED GLOBAL)
    set_property(
    TARGET rootserver-${overlay}
    PROPERTY IMPORTED_LOCATION "${CMAKE_CURRENT_BINARY_DIR}/rootserver"
    )
    add_dependencies(rootserver-${overlay} seL4-${overlay})

endforeach()

set(ElfLoaderNumNodes 4)

if((NOT Sel4testAllowSettingsOverride) AND (KernelArchARM OR KernelArchRiscV))
    # Elfloader settings that correspond to how Data61 sets its boards up.
    ApplyData61ElfLoaderSettings(${KernelPlatform} ${KernelSel4Arch})
endif()
elfloader_import_project()

add_subdirectory(apps/sel4test-driver)

include(simulation)
if(KernelSel4ArchX86_64)
    SetSimulationScriptProperty(MEM_SIZE "3G")
endif()
if(KernelPlatformQEMUArmVirt)
    SetSimulationScriptProperty(MEM_SIZE "2G")
endif()

GenerateSimulateScript()
