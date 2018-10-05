cmake_minimum_required(VERSION 3.1.0)
cmake_policy(SET CMP0043 OLD)

include(FindPkgConfig)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)

project(perseus-sdr)

find_package(LibUSB)

add_definitions(-DHAVE_CONFIG_H)

set(perseus_SOURCES
    fifo.c
    fpga_data.c
    perseuserr.c
    perseusfx2.c
    perseus-in.c
    perseus-sdr.c
)

set(perseus_HEADERS
    fifo.h
    fpga_data.h
    perseuserr.h
    perseusfx2.h
    perseus-in.h
    perseus-sdr.h
)

include_directories(
    .
    ${CMAKE_CURRENT_BINARY_DIR}
    ${LIBUSB_INCLUDE_DIR}    
    ${LIBPERSEUSSRC}
)

add_definitions(-DQT_SHARED)

add_custom_command(OUTPUT fpga_data.c 
                   COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/generate_fpga_code.sh ./ > ${CMAKE_CURRENT_BINARY_DIR}/fpga_data.c 
                   WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/)
                     
add_library(perseus-sdr SHARED
    ${perseus_SOURCES}
)

target_link_libraries(perseus-sdr
    ${LIBUSB_LIBRARIES}
)

install(FILES perseuserr.h  perseusfx2.h  perseus-in.h  perseus-sdr.h DESTINATION include)
install(TARGETS perseus-sdr DESTINATION lib)