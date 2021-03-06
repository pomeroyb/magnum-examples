#
#   This file is part of Magnum.
#
#   Original authors — credit is appreciated but not required:
#
#       2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019 —
#           Vladimír Vondruš <mosra@centrum.cz>
#       2017, 2018 — Jonathan Hale <squareys@googlemail.com>
#
#   This is free and unencumbered software released into the public domain.
#
#   Anyone is free to copy, modify, publish, use, compile, sell, or distribute
#   this software, either in source code form or as a compiled binary, for any
#   purpose, commercial or non-commercial, and by any means.
#
#   In jurisdictions that recognize copyright laws, the author or authors of
#   this software dedicate any and all copyright interest in the software to
#   the public domain. We make this dedication for the benefit of the public
#   at large and to the detriment of our heirs and successors. We intend this
#   dedication to be an overt act of relinquishment in perpetuity of all
#   present and future rights to this software under copyright law.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#   THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
#   IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
#   CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#

cmake_minimum_required(VERSION 3.4)

project(MagnumWebVrExample CXX)

# Add module path in case this is project root
if(PROJECT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/../../modules/" ${CMAKE_MODULE_PATH})
endif()

include(CheckSymbolExists)
check_symbol_exists(EMSCRIPTEN_VR_API_VERSION "emscripten/vr.h" HAS_WEBVR_1_1)
if(NOT HAS_WEBVR_1_1)
    message(ERROR "webvr-example requires WebVR 1.1 API in emscripten.")
endif()

# Closure compiler is not happy about WebVR, complaining about sourceStr and
# sourceStrPtr that probably got mangled by accident. Let's disable it for now.
string(REPLACE "--closure 1" "" CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE}")

find_package(Magnum REQUIRED
    GL
    MeshTools
    Primitives
    Shaders
    EmscriptenApplication)

set_directory_properties(PROPERTIES CORRADE_USE_PEDANTIC_FLAGS ON)

add_executable(magnum-webvr
    WebVrExample.cpp)
target_link_libraries(magnum-webvr PRIVATE
    Magnum::Application
    Magnum::GL
    Magnum::Magnum
    Magnum::MeshTools
    Magnum::Primitives
    Magnum::Shaders)

# Copy the boilerplate next to the executable so it can be run directly from
# the build dir; provide an install target as well
add_custom_command(TARGET magnum-webvr POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${MAGNUM_EMSCRIPTENAPPLICATION_JS}
        ${MAGNUM_WEBAPPLICATION_CSS}
        $<TARGET_FILE_DIR:magnum-webvr>
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_CURRENT_SOURCE_DIR}/webvr.html
        $<TARGET_FILE_DIR:magnum-webvr>/magnum-webvr.html)

install(FILES webvr.html DESTINATION ${MAGNUM_DEPLOY_PREFIX}/webvr RENAME index.html)
install(TARGETS magnum-webvr DESTINATION ${MAGNUM_DEPLOY_PREFIX}/webvr)
install(FILES
    ${MAGNUM_EMSCRIPTENAPPLICATION_JS}
    ${MAGNUM_WEBAPPLICATION_CSS}
    DESTINATION ${MAGNUM_DEPLOY_PREFIX}/webvr)
install(FILES
    $<TARGET_FILE_DIR:magnum-webvr>/magnum-webvr.js.mem
    $<TARGET_FILE_DIR:magnum-webvr>/magnum-webvr.wasm
    DESTINATION ${MAGNUM_DEPLOY_PREFIX}/webvr OPTIONAL)
