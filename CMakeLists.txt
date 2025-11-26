cmake_minimum_required(VERSION 3.10)
project(automix)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD 99)

# Include directories
include_directories(src lib lib/qm-dsp lib/vamp-plugin-sdk lib/pugixml/src)

# Find FFmpeg
find_package(PkgConfig REQUIRED)
pkg_check_modules(FFMPEG REQUIRED libavformat libavcodec libavutil)
include_directories(${FFMPEG_INCLUDE_DIRS})
link_directories(${FFMPEG_LIBRARY_DIRS})

# Build pugixml
add_subdirectory(lib/pugixml)

# Build libsamplerate
add_subdirectory(lib/libsamplerate)

# Build vamp-plugin-sdk
include(ExternalProject)
ExternalProject_Add(vamp-sdk
  SOURCE_DIR ${CMAKE_SOURCE_DIR}/lib/vamp-plugin-sdk
  CONFIGURE_COMMAND ${CMAKE_SOURCE_DIR}/lib/vamp-plugin-sdk/configure --disable-programs > /dev/null
  BUILD_COMMAND make
  INSTALL_COMMAND ""
  BUILD_IN_SOURCE 1
)

# Build qm-dsp
ExternalProject_Add(qm-dsp
  SOURCE_DIR ${CMAKE_SOURCE_DIR}/lib/qm-dsp
  CONFIGURE_COMMAND ""
  BUILD_COMMAND make -f build/linux/Makefile.linux64
  INSTALL_COMMAND ""
  BUILD_IN_SOURCE 1
)

# Build fidlib
add_library(fidlib OBJECT lib/fidlib/fidlib.c)
target_compile_definitions(fidlib PRIVATE -DT_LINUX)

# Main executable
file(GLOB SOURCES src/*.cpp src/qm/*.cpp)
add_executable(automix ${SOURCES} $<TARGET_OBJECTS:fidlib>)

# Link libraries
target_link_libraries(automix pugixml-static samplerate ${FFMPEG_LIBRARIES} pthread m)

# Dependencies
add_dependencies(automix vamp-sdk qm-dsp)

# Link static libs
target_link_libraries(automix ${CMAKE_SOURCE_DIR}/lib/qm-dsp/libqm-dsp.a ${CMAKE_SOURCE_DIR}/lib/vamp-plugin-sdk/libvamp-sdk.a)