# Seashell
# Copyright (C) 2012-2015 The Seashell Maintainers
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# See also 'ADDITIONAL TERMS' at the end of the included LICENSE file.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
cmake_minimum_required(VERSION 2.6)
project(seashell-crt)

# Get the list of C files in the runtime
file(GLOB_RECURSE CRT_SOURCES RELATIVE ${CMAKE_SOURCE_DIR} *.c)
file(GLOB_RECURSE CRT_FULL_SOURCES *.c)
file(GLOB_RECURSE CRT_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/libc/include/*.h)
file(GLOB_RECURSE CLANG_HEADERS ${CMAKE_SOURCE_DIR}/lib/clang/lib/Headers/*.h)

set(CRT_ARG_SOURCES)
foreach(file ${CRT_SOURCES})
  list(APPEND CRT_ARG_SOURCES "/source/${file}")
endforeach()

find_program(python NAMES python)

add_custom_target(crt-link-mem-files ALL
  COMMAND ln -sf ../seashell-clang.js.mem "${CMAKE_CURRENT_BINARY_DIR}")

# Build the runtime
add_custom_command(OUTPUT seashell-crt.bc
  COMMAND ${node} ${CMAKE_CURRENT_SOURCE_DIR}/build-crt.js ${CMAKE_BINARY_DIR}/compiler/seashell-clang.js ${CMAKE_CURRENT_BINARY_DIR}/seashell-crt.bc ${CRT_ARG_SOURCES}
  DEPENDS seashell-clang ${CMAKE_CURRENT_SOURCE_DIR}/build-crt.js ${CRT_FULL_SOURCES} crt-link-mem-files ${CMAKE_CURRENT_SOURCE_DIR}/platform/seashell-runtime.h)

add_custom_command(OUTPUT crt-headers.js
  COMMAND ${python} ${EMSCRIPTEN_PRELOAD} crt-headers.data --preload ${CMAKE_SOURCE_DIR}/lib/clang/lib/Headers@/clang-include/ --preload ${CMAKE_CURRENT_SOURCE_DIR}/libc/include@/include/ --preload ${CMAKE_CURRENT_SOURCE_DIR}/platform/seashell-runtime.h@/include/seashell-runtime.h --js-output=crt-headers.js
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/platform/seashell-runtime.h ${CRT_HEADERS} ${CLANG_HEADERS})

add_custom_target(seashell-crt ALL
  DEPENDS seashell-crt.bc crt-headers.js)

INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/seashell-crt.bc ${CMAKE_CURRENT_BINARY_DIR}/crt-headers.js ${CMAKE_CURRENT_BINARY_DIR}/crt-headers.data
        DESTINATION bin/)
