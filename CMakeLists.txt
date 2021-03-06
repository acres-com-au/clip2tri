# Minimum required version of CMake
cmake_minimum_required(VERSION 2.8.8)

# Set CMake project name
project(CLIP2TRI)
set (CMAKE_CXX_STANDARD 11)

set (VERSION 1.0.0-1)

# No creating project files in-source.  Bad, evil things happen
string(COMPARE EQUAL "${CMAKE_SOURCE_DIR}" "${CMAKE_BINARY_DIR}" _insource)
if (_insource)
        message(SEND_ERROR "No building in-source.  You must run CMake from the \"build\" directory.")
        message(FATAL_ERROR "Remove the file CMakeCache.txt in ${CMAKE_SOURCE_DIR} first.")
endif()


if(NOT DEFINED BUILD_SHARED_LIBS)
  set(BUILD_SHARED_LIBS ON)
endif(NOT DEFINED BUILD_SHARED_LIBS)
if(NOT DEFINED BUILD_STATIC_LIBS)
  set(BUILD_STATIC_LIBS ON)
endif(NOT DEFINED BUILD_STATIC_LIBS)

include(CheckLibraryExists)
CHECK_LIBRARY_EXISTS(m cos "" HAVE_M_LIB)
if(HAVE_M_LIB)
  set(M_LIBRARY "m")
endif(HAVE_M_LIB)

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

if(NOT DEFINED BIN_DIR)
  set(BIN_DIR bin)
endif(NOT DEFINED BIN_DIR)

if(NOT DEFINED LIB_DIR)
  set(LIB_DIR lib)
endif(NOT DEFINED LIB_DIR)

add_subdirectory(poly2tri)

if(MSVC)
  add_definitions("-DP2T_DLL_IMPORTS")
endif(MSVC)

if(BUILD_SHARED_LIBS)
  add_library(clip2tri SHARED clip2tri/clip2tri.cpp)
  target_link_libraries(clip2tri poly2tri polyclipping ${M_LIBRARY})
  install(TARGETS clip2tri
    RUNTIME DESTINATION ${BIN_DIR}
    LIBRARY DESTINATION ${LIB_DIR}
    ARCHIVE DESTINATION ${LIB_DIR})
endif(BUILD_SHARED_LIBS)

if(BUILD_STATIC_LIBS)
  add_library(clip2tri-static STATIC clip2tri/clip2tri.cpp)
  target_link_libraries(clip2tri-static poly2tri-static polyclipping ${M_LIBRARY})
  install(TARGETS clip2tri-static
    RUNTIME DESTINATION ${BIN_DIR}
    LIBRARY DESTINATION ${LIB_DIR}
    ARCHIVE DESTINATION ${LIB_DIR})
endif(BUILD_STATIC_LIBS)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/clip2tri.pc.cmakein
               ${CMAKE_CURRENT_BINARY_DIR}/clip2tri.pc @ONLY)

set(INSTALL_INC_DIR include/clip2tri)

# Install header files
install(DIRECTORY clip2tri/
        DESTINATION ${INSTALL_INC_DIR}
        FILES_MATCHING PATTERN "*.h"
)

# Install pkgconfig file
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/clip2tri.pc DESTINATION ${LIB_DIR}/pkgconfig)
