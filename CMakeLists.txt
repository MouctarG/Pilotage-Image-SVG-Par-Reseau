cmake_minimum_required(VERSION 3.10)
project(svg_pilotable)

set(CMAKE_CXX_STANDARD 17)

include(FindPkgConfig)
include(FindThreads)

pkg_check_modules(GTK REQUIRED gtk+-3.0)
pkg_check_modules(DRAWING REQUIRED cairo librsvg-2.0)

find_path(TINYXML_INCLUDE_DIR tinyxml2.h)
find_library(TINYXML2 NAMES tinyxml2)

find_path(CBOR_INCLUDE_DIR cbor.h)
find_library(CBOR NAMES cbor libcbor)

message(${CBOR_INCLUDE_DIR})

include_directories(
        main/include

        ${GTK_INCLUDE_DIRS}
        ${DRAWING_INCLUDE_DIRS}
        ${TINYXML_INCLUDE_DIR}
        ${CBOR_INCLUDE_DIR}
)

add_executable(
        svgserver

        main/serversvg.cpp
        main/src/DynamicSVG.cpp
        main/src/ServerSVGHandler.cpp
        main/src/cborUtils.cpp
        main/src/Driven.cpp
        main/src/Color.cpp
        main/include/CustomErrors.hpp
)

target_link_libraries(
        svgserver

        ${GTK_LDFLAGS}
        ${DRAWING_LDFLAGS}
        ${CBOR}
        ${TINYXML2}
        ${CMAKE_THREAD_LIBS_INIT}
)

add_executable(
        svgclientGUI

        main/clientSvgGraphique.cpp
        main/src/RemoteSVGHandler.cpp
        main/src/cborUtils.cpp
        main/include/CustomErrors.hpp
)

target_link_libraries(
        svgclientGUI

        ${CBOR}
        ${GTK_LDFLAGS}
)

add_executable(
        svgclientCLI

        main/svgclientCLI.cpp
        main/src/RemoteSVGHandler.cpp
        main/src/cborUtils.cpp
        main/include/CustomErrors.hpp
)

target_link_libraries(
        svgclientCLI

        ${CBOR}
)