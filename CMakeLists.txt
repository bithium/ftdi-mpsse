# =======================================================
#
#   @file       CMakeLists.txt
#
#   CMake configuration file for FTDI MPSSE libraries.
#
#   @author     Filipe Alves <filipe.alves@bithium.com>
#
#   @version    0.1.0
#
#   @copyright  Copyright &copy; &nbsp; 2015 Bithium S.A.
#
# =======================================================

cmake_minimum_required (VERSION 3.5)

project (ftdi-mpsse C)

# ==============================================================================
# Options
# ==============================================================================

option(FTDI_MPSSE_SHARED "Build with shared library support." On)
option(FTDI_MPSSE_SPI    "Build SPI libraries" On)
option(FTDI_MPSSE_I2C    "Build I2c libraries" On)
option(FTDI_MPSSE_SAMPLES "Build samples." Off)

# ==============================================================================
# Configuration
# ==============================================================================

string(TOLOWER ${CMAKE_SYSTEM_NAME} _system)

if(${CMAKE_SIZEOF_VOID_P} EQUAL 4)
   set(_arch "i386")
elseif(${CMAKE_SIZEOF_VOID_P} EQUAL 8)
   if(_system STREQUAL "Windows")
      set(_i2c_support False)
      set(_arch "x64")
   else()
      set(_arch "x86_64")
   endif()
endif()

if(${FTDI_MPSSE_SHARED})
  set(_sample_prefix "dynamic")
  set(_lib_suffix ${CMAKE_SHARED_LIBRARY_SUFFIX})
else()
  set(_sample_prefix "static")
  set(_lib_suffix ${CMAKE_STATIC_LIBRARY_SUFFIX})
endif()

function(ftdi_library _target)
  cmake_parse_arguments(_LIB "NO_PREFIX" "TYPE;NAME;PREFIX;SUFFIX" nop2 ${ARGN})
  if(NOT _LIB_NAME)
    set(_LIB_NAME ${_target})
  endif()
  if(NOT _LIB_TYPE)
    set(_LIB_TYPE ${_target})
  endif()
  if(NOT _LIB_PREFIX)
    set(_LIB_PREFIX ${CMAKE_SHARED_LIBRARY_PREFIX})
  endif()
  if(NOT _LIB_SUFFIX)
    set(_LIB_SUFFIX ${CMAKE_SHARED_LIBRARY_SUFFIX})
  endif()

  if(${_LIB_NO_PREFIX})
    set(_LIB_PREFIX "")
  endif()

  set(_lib_path "${PROJECT_SOURCE_DIR}/lib/${_LIB_TYPE}/${_system}/${_arch}/${_LIB_PREFIX}${_LIB_NAME}")
  if(${FTDI_MPSSE_SHARED})
    add_library(${_target} SHARED IMPORTED)
  else()
    add_library(${_target} STATIC IMPORTED)
  endif()

  set(_lib_path "${_lib_path}${_LIB_SUFFIX}")
  if(NOT EXISTS ${_lib_path})
    message(FATAL_ERROR "Could not find file : ${_lib_path}")
  endif()
  set_property(TARGET ${_target} PROPERTY IMPORTED_LOCATION ${_lib_path})
  if(NOT UNIX)
    set(_lib_path "${_lib_path}${CMAKE_LINK_LIBRARY_SUFFIX}")
    if(EXISTS "${_lib_path}")
      set_property(TARGET ${_target} PROPERTY IMPORTED_IMPLIB "${_lib_path}")
    else()
      message(FATAL_ERROR "Could not find file : ${_lib_path}")
    endif()
  endif()
endfunction()

macro(mpsse_library _type)
  ftdi_library(ftdi-mpsse-${_type}-impl TYPE ${_type} NAME "MPSSE")
  add_library(ftdi-mpsse-${_type} INTERFACE)
  target_link_libraries(ftdi-mpsse-${_type} INTERFACE ftdi-mpsse-${_type}-impl ftdi-mpsse ${CMAKE_DL_LIBS})
endmacro()

# We always require the shared version of the ftd2xx library.
set(_ftd2xx_path ${PROJECT_SOURCE_DIR}/lib/ftd2xx/${_system}/${_arch}/)
set(_ftd2xx_name "${CMAKE_SHARED_LIBRARY_PREFIX}ftd2xx${CMAKE_SHARED_LIBRARY_SUFFIX}")
set(_ftd2xx_major 1)
set(_ftd2xx_minor 4)
set(_ftd2xx_patch 6)

set(_ftd2xx_lib    "${_ftd2xx_name}.${_ftd2xx_major}.${_ftd2xx_minor}.${_ftd2xx_patch}")
set(_ftd2xx_link_1 "${_ftd2xx_name}.${_ftd2xx_major}.${_ftd2xx_minor}")
set(_ftd2xx_link_2 "${_ftd2xx_name}")

if(UNIX)
  execute_process(COMMAND ${CMAKE_COMMAND} -E create_symlink ${_ftd2xx_lib} ${_ftd2xx_link_1} WORKING_DIRECTORY ${_ftd2xx_path})
  execute_process(COMMAND ${CMAKE_COMMAND} -E create_symlink ${_ftd2xx_lib} ${_ftd2xx_link_2} WORKING_DIRECTORY ${_ftd2xx_path})
  ftdi_library(ftd2xx SUFFIX ${CMAKE_SHARED_LIBRARY_SUFFIX})
else()
  ftdi_library(ftd2xx SUFFIX ${CMAKE_SHARED_LIBRARY_SUFFIX} NO_PREFIX)
endif()

add_library(ftdi-mpsse INTERFACE)
target_include_directories(ftdi-mpsse INTERFACE include include/${_system})
target_link_libraries(ftdi-mpsse INTERFACE ftd2xx "-Wl,-rpath=${_ftd2xx_path}")

if(${FTDI_MPSSE_SPI})
  mpsse_library("spi")
endif()

if(${FTDI_MPSSE_I2C})
  mpsse_library("i2c")
endif()

# ==============================================================================
# Package
# ==============================================================================

get_target_property(FTDI_MPSSE_INCLUDE_DIRS ftdi-mpsse INTERFACE_INCLUDE_DIRECTORIES)
set(FTDI_MPSSE_INCLUDE_DIRS ${FTDI_MPSSE_INCLUDE_DIRS} CACHE INTERNAL "Pretend to have package ftdi-mpsse" FORCE)

# ==============================================================================
# Samples
# ==============================================================================

if(${FTDI_MPSSE_SAMPLES})
  if(${FTDI_MPSSE_I2C})
    add_executable(i2c-sample samples/I2C/sample-${_sample_prefix}.c)
    target_link_libraries(i2c-sample ftdi-mpsse-i2c)
  endif()

  if(${FTDI_MPSSE_SPI})
    add_executable(spi-sample samples/SPI/sample-${_sample_prefix}.c)
    target_link_libraries(spi-sample ftdi-mpsse-spi)
  endif()
endif()

# -*- mode: cmake-mode -*-
