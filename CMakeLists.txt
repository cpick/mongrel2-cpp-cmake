# author: Chris Pick <mongrel2@chirspick.com>
# license:
#
# Copyright 2010 Chris Pick. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
#    1. Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#
#    2. Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in the
#       documentation and/or other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY CHRIS PICK ``AS IS'' AND ANY EXPRESS OR IMPLIED
# WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
# EVENT SHALL CHRIS PICK OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
# INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT
# NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
# THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
# The views and conclusions contained in the software and documentation are
# those of the authors and should not be interpreted as representing official
# policies, either expressed or implied, of Chris Pick.

project(mongrel2-cpp
    "CXX"
    )


# settings


cmake_minimum_required(VERSION
    "2.8"
    )


# dependencies


# zmq

set(ZMQ_NAME
    "zmq"
    )
set(ZMQ_LIBNAME
    "lib${ZMQ_NAME}"
    )
set(ZMQ_PACKAGE
    "${ZMQ_LIBNAME}1"
    )
set(ZMQ_DEV_PACKAGE
    "${ZMQ_LIBNAME}-dev"
    )

find_package(PkgConfig)
pkg_check_modules(PC_ZMQ
    QUIET
    ${ZMQ_LIBNAME}
    )

set(ZMQ_DEFINITIONS
    ${PC_ZMQ_CFLAGS_OTHER}
    )

find_path(ZMQ_INCLUDE_DIR "${ZMQ_NAME}.hpp"
    HINTS ${PC_ZMQ_INCLUDEDIR} ${PC_ZMQ_INCLUDE_DIRS}
    )

find_library(ZMQ_LIBRARY
    NAMES ${ZMQ_NAME} ${ZMQ_LIBNAME}
    HINTS ${PC_ZMQ_LIBDIR} ${PC_ZMQ_LIBRARY_DIRS}
    )

set(ZMQ_LIBRARIES
    ${ZMQ_LIBRARY}
    )
set(ZMQ_INCLUDE_DIRS
    ${ZMQ_INCLUDE_DIR}
    )

mark_as_advanced(
    ZMQ_INCLUDE_DIR
    ZMQ_LIBRARY
    )

# json

set(JSON_NAME
    "json"
    )
set(JSON_LIBNAME
    "lib${JSON_NAME}"
    )
set(JSON_PACKAGE
    "${JSON_LIBNAME}0"
    )
set(JSON_DEV_PACKAGE
    "${JSON_PACKAGE}-dev"
    )

find_package(PkgConfig)
pkg_check_modules(PC_JSON
    QUIET
    ${JSON_PACKAGE}
    )

set(JSON_DEFINITIONS
    ${PC_JSON_CFLAGS_OTHER}
    )

find_path(JSON_INCLUDE_DIR "${JSON_NAME}/${JSON_NAME}.h"
    HINTS ${PC_JSON_INCLUDEDIR} ${PC_JSON_INCLUDE_DIRS}
    )

find_library(JSON_LIBRARY
    NAMES ${JSON_NAME} ${JSON_LIBNAME}
    HINTS ${PC_JSON_LIBDIR} ${PC_JSON_LIBRARY_DIRS}
    )

set(JSON_LIBRARIES
    ${JSON_LIBRARY}
    )
set(JSON_INCLUDE_DIRS
    ${JSON_INCLUDE_DIR}
    )

mark_as_advanced(
    JSON_INCLUDE_DIR
    JSON_LIBRARY
    )

# m2pp

set(M2PP_PROJECT
    "m2pp"
    )

set(M2PP_ANAME
    "lib${M2PP_PROJECT}.a"
    )

# TODO Debug vs Release?
# TODO definitions ${ZMQ_DEFINITIONS} ${JSON_DEFINITIONS}
include(ExternalProject)
ExternalProject_Add(${M2PP_PROJECT}
    PREFIX ${M2PP_PROJECT}
    URL "${CMAKE_CURRENT_SOURCE_DIR}/mongrel2-cpp"
    CONFIGURE_COMMAND ""
    BUILD_IN_SOURCE 1
    BUILD_COMMAND make "${M2PP_ANAME}"
    INSTALL_COMMAND make "DESTDIR=<INSTALL_DIR>" install
    )

ExternalProject_Get_Property(${M2PP_PROJECT}
        "INSTALL_DIR")

# Custom command that explicitly "creates" the library file, this is needed by
# ninja to track where the file comes from.
# TODO: When we can require on cmake >=3.2 replace this phone custom command
# with a BYPRODUCTS section in the above ExternalProject_Add.

set(M2PP_PREFIX
    "${INSTALL_DIR}/usr/local"
    )

set(M2PP_LIBRARY
    "${M2PP_PREFIX}/lib/${M2PP_ANAME}"
    )

add_custom_command(OUTPUT ${M2PP_LIBRARY}
    COMMAND ${CMAKE_COMMAND} -E touch_nocreate ${M2PP_LIBRARY}
    MAIN_DEPENDENCY ${M2PP_PROJECT}
    )

set(M2PP_DUMMY
    "m2pp-dummy"
    )
add_custom_target(${M2PP_DUMMY}
    SOURCES ${M2PP_LIBRARY}
    )

# target

set(M2PP_LIBRARIES
    "m2pp-lib"
    CACHE INTERNAL "m2pp library name"
    )

add_library(${M2PP_LIBRARIES}
    STATIC IMPORTED
    GLOBAL
    )

# dependencies

add_dependencies(${M2PP_LIBRARIES}
    ${M2PP_DUMMY}
    )

set_property(TARGET ${M2PP_LIBRARIES}
    PROPERTY IMPORTED_LOCATION ${M2PP_LIBRARY}
    )

# includes

set(M2PP_INCLUDE_DIRS
    "${M2PP_PREFIX}/include"
    )

file(MAKE_DIRECTORY
    ${M2PP_INCLUDE_DIRS}
    )

set_property(TARGET ${M2PP_LIBRARIES}
    PROPERTY INTERFACE_INCLUDE_DIRECTORIES
        ${M2PP_INCLUDE_DIRS}
        ${ZMQ_INCLUDE_DIRS}
        ${JSON_INCLUDE_DIRS}
    )

# libraries

set_property(TARGET ${M2PP_LIBRARIES}
    PROPERTY IMPORTED_LINK_INTERFACE_LIBRARIES
        ${ZMQ_LIBRARIES}
        ${JSON_LIBRARIES}
    )
