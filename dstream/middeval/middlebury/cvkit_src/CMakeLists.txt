# This file is part of the Computer Vision Toolkit (cvkit).
#
# Author: Heiko Hirschmueller
#
# Copyright (c) 2014, Institute of Robotics and Mechatronics, German Aerospace Center
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice,
# this list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its contributors
# may be used to endorse or promote products derived from this software without
# specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required (VERSION 2.8.9)

project(cvkit)

include_directories(${CMAKE_SOURCE_DIR})

# settings for NSIS (Windows) install package

set(CPACK_GENERATOR NSIS)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Computer Vision Toolkit")
set(CPACK_PACKAGE_VENDOR "Institute of Robotics and Mechatronics, German Aerospace Center")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.txt")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "6")
set(CPACK_PACKAGE_VERSION_PATCH "6")
set(CVKIT_VERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH})
set(CPACK_NSIS_INSTALL_ROOT "C:")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "cvkit")

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

set(CPACK_NSIS_EXTRA_INSTALL_COMMANDS "
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\plyv.exe' '.ply' 'cvkit.plyv.ply'
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\sv.exe' '.pfm' 'cvkit.sv.pfm'
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\sv.exe' '.pgm' 'cvkit.sv.pgm'
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\sv.exe' '.ppm' 'cvkit.sv.ppm'
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\sv.exe' '.tif' 'cvkit.sv.tif'
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\sv.exe' '.jpg' 'cvkit.sv.jpg'
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\sv.exe' '.png' 'cvkit.sv.png'
  \\\${registerExtension} '\\\$INSTDIR\\\\bin\\\\sv.exe' '.vrt' 'cvkit.sv.vrt'
  System::Call 'shell32.dll::SHChangeNotify(i, i, i, i) v (0x08000000, 0, 0, 0)'
")

set(CPACK_NSIS_CREATE_ICONS_EXTRA "
  CreateShortCut '\\\$SMPROGRAMS\\\\\$STARTMENU_FOLDER\\\\Example.lnk' '\\\$INSTDIR\\\\example'
  CreateShortCut '\\\$SMPROGRAMS\\\\\$STARTMENU_FOLDER\\\\Readme.lnk' '\\\$INSTDIR\\\\doc\\\\README.txt'
  CreateShortCut '\\\$SMPROGRAMS\\\\\$STARTMENU_FOLDER\\\\Usage.lnk' '\\\$INSTDIR\\\\doc\\\\USAGE.txt'
")

set(CPACK_NSIS_EXTRA_UNINSTALL_COMMANDS "
  \\\${unregisterExtension} '.ply' 'cvkit.plyv.ply'
  \\\${unregisterExtension} '.pfm' 'cvkit.sv.pfm'
  \\\${unregisterExtension} '.pgm' 'cvkit.sv.pgm'
  \\\${unregisterExtension} '.ppm' 'cvkit.sv.ppm'
  \\\${unregisterExtension} '.tif' 'cvkit.sv.tif'
  \\\${unregisterExtension} '.jpg' 'cvkit.sv.jpg'
  \\\${unregisterExtension} '.png' 'cvkit.sv.png'
  \\\${unregisterExtension} '.vrt' 'cvkit.sv.vrt'
  System::Call 'shell32.dll::SHChangeNotify(i, i, i, i) v (0x08000000, 0, 0, 0)'
")

set(CPACK_NSIS_DELETE_ICONS_EXTRA "
  Delete '\\\$SMPROGRAMS\\\\\$MUI_TEMP\\\\Readme.lnk'
  Delete '\\\$SMPROGRAMS\\\\\$MUI_TEMP\\\\Usage.lnk'
  Delete '\\\$SMPROGRAMS\\\\\$MUI_TEMP\\\\Example.lnk'
")

include(CPack)

# set default build type to release

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RELEASE CACHE STRING "Build type: DEBUG or RELEASE" FORCE)
endif ()

# choose to install header and libs

set(INSTALL_DEVELOPMENT 0 CACHE BOOL "Install header and libs for development with cvkit")

# check for optional external packages

find_package(JPEG)
find_package(PNG)
find_package(GDAL)
find_package(OpenGL)
find_package(GLUT)
find_package(GLEW)

if (NOT WIN32 AND NOT CYGWIN)
  find_package(X11)
endif ()

# - Optionally compile as C++11 -

set(USE_CPP11 0 CACHE BOOL "Compile as C++11 (and use standard threads)")

if (USE_CPP11)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -DINCLUDE_CPP11")
endif ()

# - Optional pthread support -

find_package(Threads)

if (CMAKE_USE_PTHREADS_INIT)
  set(INCLUDE_PTHREADS 1 CACHE BOOL "Use pthreads")

  if (INCLUDE_PTHREADS)
    add_definitions(-DINCLUDE_PTHREADS)
  endif ()
else ()
  set(INCLUDE_PTHREADS 0 CACHE BOOL "Use pthreads" FORCE)
endif ()

# - Standard definitions -

add_definitions(-Wall)

#set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# build individual parts

add_subdirectory(gutil)
add_subdirectory(gmath)
add_subdirectory(gimage)
add_subdirectory(bgui)
add_subdirectory(gvr)

# build excecutables

add_subdirectory(tools)

# add examples

add_subdirectory(example)

# install cmake config files in lib install directory

if (INSTALL_DEVELOPMENT)
  export(TARGETS gutil gmath gimage bgui gvr
    FILE "${PROJECT_BINARY_DIR}/CVKITTargets.cmake")
  
  set(CONF_INCLUDE_DIRS "\${CVKIT_CMAKE_DIR}/../../include")
  
  configure_file(cmake/CVKITConfig.cmake.in
    "${PROJECT_BINARY_DIR}/CVKITConfig.cmake" @ONLY)
  
  configure_file(cmake/CVKITConfigVersion.cmake.in
    "${PROJECT_BINARY_DIR}/CVKITConfigVersion.cmake" @ONLY)
  
  install(FILES
    "${PROJECT_BINARY_DIR}/CVKITConfig.cmake"
    "${PROJECT_BINARY_DIR}/CVKITConfigVersion.cmake"
    DESTINATION lib/cvkit)
  
  install(EXPORT CVKITTargets DESTINATION lib/cvkit)
endif ()
