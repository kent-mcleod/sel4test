cmake_minimum_required(VERSION 3.7.2)

include(../settings.cmake)
project(seL4 C CXX ASM)

# Set the roottask CNode size to be large enough for DTB, timer caps, etc
# Note that this works for the platforms that we have at the moment, and may
# need to be increased in the future
set(KernelRootCNodeSizeBits 13 CACHE INTERNAL "")

# Set our custom domain schedule
# set(KernelDomainSchedule "${CMAKE_CURRENT_LIST_DIR}/domain_schedule.c" CACHE INTERNAL "")
sel4_import_kernel()
