#***************************************************************************
#                                  _   _ ____  _
#  Project                     ___| | | |  _ \| |
#                             / __| | | | |_) | |
#                            | (__| |_| |  _ <| |___
#                             \___|\___/|_| \_\_____|
#
# Copyright (C) Daniel Stenberg, <daniel@haxx.se>, et al.
#
# This software is licensed as described in the file COPYING, which
# you should have received as part of this distribution. The terms
# are also available at https://curl.se/docs/copyright.html.
#
# You may opt to use, copy, modify, merge, publish, distribute and/or sell
# copies of the Software, and permit persons to whom the Software is
# furnished to do so, under the terms of the COPYING file.
#
# This software is distributed on an "AS IS" basis, WITHOUT WARRANTY OF ANY
# KIND, either express or implied.
#
# SPDX-License-Identifier: curl
#
###########################################################################

# Get BUNDLE, BUNDLE_SRC, FIRSTFILES, UTILS, CURLX_SRCS, TESTFILES variables
curl_transform_makefile_inc("Makefile.inc" "${CMAKE_CURRENT_BINARY_DIR}/Makefile.inc.cmake")
include("${CMAKE_CURRENT_BINARY_DIR}/Makefile.inc.cmake")

set(_bundle_extra "")
if(LIB_SELECTED STREQUAL LIB_SHARED)
  list(APPEND _bundle_extra ${CURLX_SRCS})  # Not exported from the libcurl shared build. Build a copy.
endif()

add_custom_command(OUTPUT "lib1521.c"
  COMMAND ${PERL_EXECUTABLE} "${CMAKE_CURRENT_SOURCE_DIR}/mk-lib1521.pl" < "${PROJECT_SOURCE_DIR}/include/curl/curl.h" "lib1521.c"
  DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/mk-lib1521.pl"
    "${PROJECT_SOURCE_DIR}/include/curl/curl.h"
  VERBATIM)

add_custom_command(OUTPUT "${BUNDLE_SRC}"
  COMMAND ${PERL_EXECUTABLE} "${PROJECT_SOURCE_DIR}/scripts/mk-unity.pl"
    --include ${UTILS} ${_bundle_extra} --test ${TESTFILES} "lib1521.c"
    ${CURL_MK_UNITY_OPTION} --srcdir "${CMAKE_CURRENT_SOURCE_DIR}" > "${BUNDLE_SRC}"
  DEPENDS
    "${PROJECT_SOURCE_DIR}/scripts/mk-unity.pl" "${CMAKE_CURRENT_SOURCE_DIR}/Makefile.inc"
    ${FIRSTFILES} ${UTILS} ${_bundle_extra} ${TESTFILES} "lib1521.c"
  VERBATIM)

add_executable(libtests EXCLUDE_FROM_ALL "${BUNDLE_SRC}")
add_dependencies(testdeps libtests)
target_include_directories(libtests PRIVATE
  "${PROJECT_BINARY_DIR}/lib"            # for "curl_config.h"
  "${PROJECT_SOURCE_DIR}/lib"            # for "curl_setup.h"
  "${PROJECT_SOURCE_DIR}/lib/curlx"      # for curlx
  "${PROJECT_SOURCE_DIR}/src"            # for "tool_binmode.h"
  "${PROJECT_SOURCE_DIR}/tests/libtest"  # to be able to build generated tests
  "${PROJECT_SOURCE_DIR}/tests/unit"     # for curlcheck.h
)
target_link_libraries(libtests ${LIB_SELECTED} ${CURL_LIBS})
set_property(TARGET libtests APPEND PROPERTY COMPILE_DEFINITIONS "${CURL_DEBUG_MACROS}" "CURL_NO_OLDIES" "CURL_DISABLE_DEPRECATION")
set_target_properties(libtests PROPERTIES OUTPUT_NAME "${BUNDLE}" PROJECT_LABEL "Test ${BUNDLE}" UNITY_BUILD OFF)
