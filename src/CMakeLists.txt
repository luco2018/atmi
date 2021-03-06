#
#MIT License 
#
#Copyright © 2016 Advanced Micro Devices, Inc.  
#
#Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software
#without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
#persons to whom the Software is furnished to do so, subject to the following conditions:
#
#The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
#THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR
#PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
#OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#

cmake_minimum_required (VERSION 2.8 FATAL_ERROR)

list ( APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules" )
include ( utils )

include (FindPackageHandleStandardArgs)

project (atmi)

macro(libatmi_runtime_say message_to_user)
  message(STATUS "ATMI: ${message_to_user}")
endmacro()

set (CMAKE_C_COMPILER g++)
# The version number.
set (atmi_VERSION_MAJOR 0)
set (atmi_VERSION_MINOR 3)

################################################################################
# Default install path to ROCM and not /usr/local
################################################################################
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set (CMAKE_INSTALL_PREFIX "/opt/rocm/libatmi" CACHE PATH "default install path" FORCE )
endif()
################################################################################
# Looking for ROCM...
################################################################################
#pkg_check_modules(ATMI_SEARCH_LIBHSA QUIET libhsa-runtime64)

find_file (
  ATMI_DEP_LIBHSA_INCLUDE
  hsa/hsa.h
  PATHS
  /opt/rocm/include
  /usr/local/include
  )
get_filename_component (ATMI_DEP_LIBHSA_INCLUDE_DIRS ${ATMI_DEP_LIBHSA_INCLUDE} DIRECTORY)

find_library (
  ATMI_DEP_LIBHSA_LIBRARIES
  libhsa-runtime64.so
  PATHS
  /opt/rocm/lib
  /usr/local/lib
  )
get_filename_component (ATMI_DEP_LIBHSA_LIBRARIES_DIRS ${ATMI_DEP_LIBHSA_LIBRARIES} DIRECTORY)

find_package_handle_standard_args(
  ATMI_DEP_LIBHSA
  DEFAULT_MSG
  ATMI_DEP_LIBHSA_LIBRARIES_DIRS
  ATMI_DEP_LIBHSA_INCLUDE_DIRS)

mark_as_advanced(
  ATMI_DEP_LIBHSA_INCLUDE_DIRS
  ATMI_DEP_LIBHSA_LIBRARIES_DIRS)
 
add_subdirectory(runtime) 
add_subdirectory(compiler) 

## Get the package version. The defaults to 0.0.1.
get_version ( "0.0.1" )

set ( BUILD_VERSION_MAJOR ${VERSION_MAJOR} )
set ( BUILD_VERSION_MINOR ${VERSION_MINOR} )
set ( BUILD_VERSION_PATCH ${VERSION_PATCH} )
set ( LIB_VERSION_STRING "${BUILD_VERSION_MAJOR}.${BUILD_VERSION_MINOR}.${BUILD_VERSION_PATCH}" )
if ( DEFINED VERSION_BUILD )
    set ( BUILD_VERSION_PATCH "${BUILD_VERSION_PATCH}-${VERSION_BUILD}" )
endif ()
set ( BUILD_VERSION_STRING "${BUILD_VERSION_MAJOR}.${BUILD_VERSION_MINOR}.${BUILD_VERSION_PATCH}" )

## Packaging directives
set ( CPACK_PACKAGE_NAME "atmi" )
set ( CPACK_PACKAGE_VENDOR "Advanced Micro Devices Inc." )
set ( CPACK_PACKAGE_VERSION_MAJOR ${BUILD_VERSION_MAJOR} )
set ( CPACK_PACKAGE_VERSION_MINOR ${BUILD_VERSION_MINOR} )
set ( CPACK_PACKAGE_VERSION_PATCH ${BUILD_VERSION_PATCH} )
set ( CPACK_PACKAGE_CONTACT "Ashwin Aji <ashwin.aji@amd.com>" )
set ( CPACK_PACKAGE_DESCRIPTION_SUMMARY "ATMI: Asynchronous Task and Memory Interface" )
set ( CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/../LICENSE.txt" )
set ( CPACK_PACKAGING_INSTALL_PREFIX /opt/rocm/atmi )
# Debian package specific variables
set ( CPACK_DEBIAN_PACKAGE_DEPENDS "hsa-rocr-dev" )
set ( CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://github.com/RadeonOpenCompute/atmi" )
set ( CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_CURRENT_SOURCE_DIR}/DEBIAN/postinst;${CMAKE_CURRENT_SOURCE_DIR}/DEBIAN/prerm"
)
set ( CPACK_DEB_COMPONENT_INSTALL ON)

## RPM package specific variables
set ( CPACK_RPM_PACKAGE_DEPENDS "hsa-rocr-dev" )
set ( CPACK_RPM_PRE_INSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/RPM/rpm_post" )
set ( CPACK_RPM_POST_UNINSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/RPM/rpm_postun" )
set ( CPACK_RPM_COMPONENT_INSTALL ON)


## Set components
set ( CPACK_COMPONENTS_ALL runtime cplugin )
set ( CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE 1 )

## Include packaging
include ( CPack )
