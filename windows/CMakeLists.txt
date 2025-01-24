# dart_vlc: A media playback library for Dart & Flutter. Based on libVLC & libVLC++.
#
# Hitesh Kumar Saini
# https://github.com/alexmercerind
# alexmercerind@gmail.com
# GNU Lesser General Public License v2.1
# 
cmake_minimum_required(VERSION 3.15)
set(CMAKE_CXX_STANDARD 17)

set(LIBVLC_VERSION "3.0.9.2")
set(PACKAGE "dart_vlc")
add_library(${PACKAGE} SHARED
  "../ffi/native/dart_vlc.cpp"
)

set(LIBVLC_BINARIES "${CMAKE_CURRENT_SOURCE_DIR}/../bin")

set(LIBVLC_ARCHIVE "${LIBVLC_BINARIES}/vlc-${LIBVLC_VERSION}.7z")
set(LIBVLCPP_ARCHIVE "${LIBVLC_BINARIES}/libvlcpp.zip")

set(LIBVLC_SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/external/vlc-${LIBVLC_VERSION}")
set(LIBVLCPP_SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/external/libvlcpp-master")

add_custom_target(LIBVLC_EXTRACT ALL)

if (NOT EXISTS "${LIBVLC_SOURCE}" AND NOT EXISTS "${LIBVLCPP_SOURCE}")
  add_custom_command(
    TARGET LIBVLC_EXTRACT PRE_BUILD
    COMMAND ${CMAKE_COMMAND} -E tar xzf \"${LIBVLC_ARCHIVE}\"
    COMMAND ${CMAKE_COMMAND} -E tar xzf \"${LIBVLCPP_ARCHIVE}\"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/external"
    DEPENDS "${LIBVLC_ARCHIVE}"
  )
endif()

include_directories(${PACKAGE} INTERFACE
  # libVLC++ Headers.
  ${LIBVLCPP_SOURCE}
  # libVLC Headers.
  ${LIBVLC_SOURCE}/sdk/include
)

target_include_directories(${PACKAGE} INTERFACE
  # DartVLC Headers.
  ${CMAKE_CURRENT_SOURCE_DIR}/../ffi/native
)
target_link_libraries("${PACKAGE}" PRIVATE
  "${LIBVLC_SOURCE}/sdk/lib/libvlc.lib"
  "${LIBVLC_SOURCE}/sdk/lib/libvlccore.lib"
)


# Not used Flutter plugin template.

set(PROJECT_NAME "dart_vlc")
set(PLUGIN_NAME "dart_vlc_plugin")
project(${PROJECT_NAME} LANGUAGES CXX)
set("${PACKAGE}" "dart_vlc_plugin")


include_directories("${PLUGIN_NAME}" INTERFACE
  "${CMAKE_CURRENT_SOURCE_DIR}/include"
)
add_library("${PLUGIN_NAME}" SHARED
  "dart_vlc_plugin.cpp"
)

apply_standard_settings("${PLUGIN_NAME}")
set_target_properties("${PLUGIN_NAME}" PROPERTIES
  CXX_VISIBILITY_PRESET hidden)
target_compile_definitions("${PLUGIN_NAME}" PRIVATE FLUTTER_PLUGIN_IMPL)
target_include_directories("${PLUGIN_NAME}" INTERFACE
  "${CMAKE_CURRENT_SOURCE_DIR}/include"
  "${CMAKE_CURRENT_SOURCE_DIR}/../dartvlc"
)

target_link_libraries("${PLUGIN_NAME}" PRIVATE
  flutter
  flutter_wrapper_plugin
)

# Add generated shared library & libVLC DLLs.

set(
  dart_vlc_bundled_libraries
  # dart_vlc.dll
  "$<TARGET_FILE:dart_vlc>"
  "${LIBVLC_SOURCE}/libvlc.dll"
  "${LIBVLC_SOURCE}/libvlccore.dll"
  "${LIBVLC_SOURCE}/plugins"
  PARENT_SCOPE
)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4244 /wd4996")
