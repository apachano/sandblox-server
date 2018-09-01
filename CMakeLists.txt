#
#	 Copyright (C) 2018 Qub³d Engine Group.
#	 All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without modification,
#  are permitted provided that the following conditions are met:
#
#  1. Redistributions of source code must retain the above copyright notice, this
#  list of conditions and the following disclaimer.
#
#  2. Redistributions in binary form must reproduce the above copyright notice,
#  this list of conditions and the following disclaimer in the documentation and/or
#  other materials provided with the distribution.
#
#  3. Neither the name of the copyright holder nor the names of its contributors
#  may be used to endorse or promote products derived from this software without
#  specific prior written permis`sion.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
#  ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
#  WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
#  DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
#  ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
#  (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
#  LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
#  ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
#  (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
#  SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#

# Use this version because it adds some options that we use.
cmake_minimum_required(VERSION 3.0.2 FATAL_ERROR)

project(blockforge-server)

# This is necessary when we need to debug CMake.
set(CMAKE_VERBOSE_MAKEFILE ON)

# Set the directory for CMake modules dedicated to finding dependencies.
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

if (CMAKE_BUILD_TYPE STREQUAL "")
  set(CMAKE_BUILD_TYPE "Debug")
endif()

# Establish fundamental compiler rules.

option(32BIT "Build 32-bit on either 32-bit or 64-bit systems" OFF)
option(64BIT "Build 64-bit on only 64-bit systems" ON)

set(WARNINGS "-Wall -Wextra")

if (${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  message(FATAL_ERROR "MSVC is not supported.\n For more information as to why, see https://phab.qub3d.org/V8")
endif()

# Global compiler flags.
if(APPLE)
        set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${WARNINGS} -std=c++1z")
        set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${WARNINGS} -O0 -std=c++1z")
else()
        set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${WARNINGS} -std=c++17")
        set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${WARNINGS} -O0 -std=c++17")
endif()

set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} ${WARNINGS} -std=c11")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${WARNINGS} -O0 -std=c11")

# Establish settings for Clang and GCC.
if (${CMAKE_CXX_COMPILER_ID} STREQUAL Clang)
  if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS "2.0")
        message(FATAL_ERROR "You need Clang to be newer than 2.0. Halting.")
  endif()
  if (LLVM)
        set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -stdlib=libc++")
        set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -stdlib=libc++")
  endif()
elseif (${CMAKE_CXX_COMPILER_ID} STREQUAL GNU)
  if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS "5")
        message(FATAL_ERROR "You need GCC to be newer than 5.x. Halting.")
  endif()
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -s")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -s")
endif()

# Detect a CPU architecture to target in order to make unrecognizable executables a thing.
if (${32BIT} STREQUAL ON)
  set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -m32")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -m32")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -m32")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -m32")
elseif (${64BIT} STREQUAL ON)
  set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -m64")
  set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -m64")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -m64")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -m64")
endif()

# Set include, library, and source directories.
set(libdeps ${CMAKE_CURRENT_LIST_DIR}/../qub3d-libdeps)
set(source_dir ${PROJECT_SOURCE_DIR}/source/src)
set(headers_root ${source_dir}/../)

set(client_sources
        ${source_dir}/server.cpp
        ${source_dir}/main.cpp
)

set(include_dirs
        ${libdeps}/cpgf/include
        ${libdeps}/pawlib/pawlib-source/include
        ${PROJECT_SOURCE_DIR}/../qub3d-engine/source/include
        ${source_dir}/../include
)
message(${include_dirs})

if (UNIX AND NOT APPLE)
        find_package(Threads REQUIRED)
        set(library_dirs
                ${PROJECT_SOURCE_DIR}/../qub3d-engine/build/libQub3d.a
                ${libdeps}/pawlib/lib/libpawlib.as
                ${libdeps}/cpgf/lib/libcpgf.a
        )
else()
        set(library_dirs
                ${PROJECT_SOURCE_DIR}/../qub3d-engine/build/libQub3d.a
                ${libdeps}/pawlib/lib/libpawlib.a
                ${libdeps}/cpgf/lib/libcpgf.a
        )
endif()# UNIX AND NOT APPLE

# Link the "Server" binary.
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
set(POSITION_INDEPENDENT_CODE True)
add_executable(sandblox-server ${client_sources})
include_directories(${include_dirs})
target_link_libraries(sandblox-server ${library_dirs})

# Install it system-wide. (Invoked by `sudo make install`)
if (UNIX)
  install(
        TARGETS sandblox-server
        RUNTIME DESTINATION /usr/bin
  )
endif()
