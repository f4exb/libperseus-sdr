cmake_minimum_required(VERSION 3.1.0)
cmake_policy(SET CMP0043 OLD)

include(FindPkgConfig)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)

include(GNUInstallDirs)
set(LIB_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}") # "lib" or "lib64"

project(perseus-sdr)

find_package(LibUSB)

add_definitions(-DHAVE_CONFIG_H)

set(perseus_SOURCES
    fpga_data.c
    perseuserr.c
    perseusfx2.c
    perseus-in.c
    perseus-sdr.c
)

set(perseus_HEADERS
    config.h
    fpga_data.h
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

add_custom_command(OUTPUT config.h
				   COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/bootstrap.sh;COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure
				   WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/)

add_custom_command(OUTPUT fpga_data.c
                   COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/generate_fpga_code.sh ./ > ${CMAKE_CURRENT_BINARY_DIR}/fpga_data.c
                   WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/)

add_library(perseus-sdr SHARED
    ${perseus_SOURCES}
)

add_custom_target(configuration DEPENDS config.h)
add_dependencies(perseus-sdr configuration)

target_link_libraries(perseus-sdr
    ${LIBUSB_LIBRARIES}
)

install(FILES perseusfx2.h  perseus-in.h  perseus-sdr.h DESTINATION include)
install(TARGETS perseus-sdr DESTINATION ${LIB_INSTALL_DIR})