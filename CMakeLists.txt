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
# Configuration
# ==============================================================================

option(FTDI_MPSSE_SHARED "Build with shared library support." On)
option(FTDI_MPSSE_SPI    "Build SPI libraries" On)
option(FTDI_MPSSE_I2C    "Build I2c libraries" On)
option(FTDI_MPSSE_SAMPLES "Build samples." Off)

# ==============================================================================
# Configuration
# ==============================================================================

string(TOLOWER ${CMAKE_SYSTEM_NAME} _system)

include(CheckTypeSize)
check_type_size (void* VOID_P_SIZE)

set(_i2c_support True)

if(${VOID_P_SIZE} EQUAL 4)
   set(_arch "i386")
elseif(${VOID_P_SIZE} EQUAL 8)
   if(_system STREQUAL "Windows")
      set(_i2c_support False)
      set(_arch "x64")
   else()
      set(_arch "x86_64")
   endif()
endif()

macro(ftdi_library type)
  set(_lib_path "${PROJECT_SOURCE_DIR}/lib/${type}/${_system}/${_arch}/libMPSSE")
  set(_lib_name ftdi-mpsse-${type}-impl)
  if(${FTDI_MPSSE_SHARED})
    set(_sample_prefix "dynamic")
    add_library(${_lib_name} SHARED IMPORTED)
    if(UNIX)
      set(_lib_ext "so")
    else()
      set(_lib_ext "dll")
      if(EXISTS "${_lib_path}.lib")
        set_property(TARGET ${_lib_name} PROPERTY IMPORTED_IMPLIB "${_lib_path}.lib")
      else()
        set(_${type}_support False)
      endif()
    endif()
  else()
    add_library(${_lib_name} STATIC IMPORTED)
    set(_sample_prefix "static")
    set(_lib_ext "a")
  endif()

  set(_lib_file "${_lib_path}.${_lib_ext}")
  # message(STATUS ${_lib_file})
  if(NOT EXISTS ${_lib_file})
    message(FATAL_ERROR "Could not find file : ${_lib_file}")
  endif()
  set_property(TARGET ${_lib_name} PROPERTY IMPORTED_LOCATION ${_lib_file})

  add_library(ftdi-mpsse-${type} INTERFACE)
  target_link_libraries(ftdi-mpsse-${type} INTERFACE ftdi-mpsse ${_lib_name})

  if(UNIX)
    target_link_libraries(ftdi-mpsse-${type} INTERFACE dl)
  endif()
endmacro()

add_library(ftdi-mpsse INTERFACE)
target_include_directories(ftdi-mpsse INTERFACE include include/${_system})

if(${FTDI_MPSSE_SPI})
  ftdi_library("spi")
endif()

if(${FTDI_MPSSE_I2C})
  ftdi_library("i2c")
  if(NOT ${_i2c_support})
    message(WARNING "\
    I2C is currently not supported on this configuration
    (Windows + Shared lib.) !")
  endif()
endif()


# ==============================================================================
# Samples
# ==============================================================================

if(${FTDI_MPSSE_SAMPLES})
  if(${FTDI_MPSSE_I2C} AND ${_i2c_support})
    add_executable(i2c-sample samples/I2C/sample-${_sample_prefix}.c)
    target_link_libraries(i2c-sample ftdi-mpsse-i2c)
  endif()

  if(${FTDI_MPSSE_SPI})
    add_executable(spi-sample samples/SPI/sample-${_sample_prefix}.c)
    target_link_libraries(spi-sample ftdi-mpsse-spi)
  endif()
endif()

# -*- mode: cmake-mode -*-
