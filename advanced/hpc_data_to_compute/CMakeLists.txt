cmake_minimum_required(VERSION 3.12...3.18 FATAL_ERROR)
if (POLICY CMP0109)
  # find_program requires permission to execute but not to read
  cmake_policy(SET CMP0109 NEW)
endif()
if (POLICY CMP0121)
  # Detect invalid indices in list()
  cmake_policy(SET CMP0121 NEW)
endif()
if (POLICY CMP0125)
  # Consistent behavior for cache variables managed by find_*()
  cmake_policy(SET CMP0125 NEW)
endif()
if (POLICY CMP0130)
  # Diagnose condition evaluation errors in while()
  cmake_policy(SET CMP0130 NEW)
endif()
if (POLICY CMP0132)
  # Consistent handling of compiler environment variables
  cmake_policy(SET CMP0132 NEW)
endif()
if (POLICY CMP0140)
  # Check validity of return() params
  cmake_policy(SET CMP0140 NEW)
endif()

set(IRODS_MINIMUM_VERSION "4.3.2")
set(IRODS_MAXIMUM_VERSION "6.0.0")
find_package(IRODS "${IRODS_MINIMUM_VERSION}...<${IRODS_MAXIMUM_VERSION}" REQUIRED)

project(irods_hpc_data_to_compute_example)

install(
  FILES
  ${CMAKE_SOURCE_DIR}/data_to_compute.re
  ${CMAKE_SOURCE_DIR}/core.py.data_to_compute
  DESTINATION etc/irods
  PERMISSIONS OWNER_READ GROUP_READ WORLD_READ
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/spawn_remote_slurm_jobs.r
  ${CMAKE_SOURCE_DIR}/rescName_from_kvpair.r
  ${CMAKE_SOURCE_DIR}/detect_thumbnails.py
  DESTINATION ${IRODS_HOME_DIRECTORY}
  PERMISSIONS OWNER_READ GROUP_READ WORLD_READ
  )


install(
  FILES
  ${CMAKE_SOURCE_DIR}/compute/common.py
  ${CMAKE_SOURCE_DIR}/compute/util.py
  ${CMAKE_SOURCE_DIR}/compute/job_params.json
  ${CMAKE_SOURCE_DIR}/compute/__init__.py
  ${CMAKE_SOURCE_DIR}/compute/irods_compute_functions
  DESTINATION ${IRODS_HOME_DIRECTORY}/compute
  PERMISSIONS OWNER_READ GROUP_READ WORLD_READ
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/compute/admin_as_rodsuser.json
  DESTINATION ${IRODS_HOME_DIRECTORY}/compute
  PERMISSIONS OWNER_READ OWNER_WRITE
  )

install(
  FILES
  ${CMAKE_SOURCE_DIR}/submit_thumbnail_job.sh
  ${CMAKE_SOURCE_DIR}/convert.SLURM
  DESTINATION ${IRODS_HOME_DIRECTORY}/msiExecCmd_bin
  PERMISSIONS OWNER_READ OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
  )

if (NOT CPACK_PACKAGING_INSTALL_PREFIX)
  set(CPACK_PACKAGING_INSTALL_PREFIX "/" CACHE STRING "Package root path. \"/\" is correct for normal package builds.." FORCE)
  message(STATUS "Setting unspecified CPACK_PACKAGING_INSTALL_PREFIX to '${CPACK_PACKAGING_INSTALL_PREFIX}'. This is the correct setting for normal builds.")
endif()


set(CPACK_PACKAGE_FILE_NAME "irods-hpc-data-to-compute-example${IRODS_PACKAGE_FILE_NAME_SUFFIX}")
set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_COMPONENT_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_COMPONENTS_GROUPING IGNORE)
set(CPACK_PACKAGE_VERSION ${IRODS_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR ${IRODS_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${IRODS_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${IRODS_VERSION_PATCH})
set(CPACK_PACKAGE_CONTACT "Renaissance Computing Institute <info@irods.org>")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "The integrated Rule-Oriented Data System")
set(CPACK_PACKAGE_VENDOR "Renaissance Computing Institute <info@irods.org>")

set(CPACK_DEB_COMPONENT_INSTALL OFF)
set(CPACK_DEBIAN_PACKAGE_SECTION "contrib/science")
set(CPACK_DEBIAN_COMPRESSION_TYPE "gzip")
set(CPACK_DEBIAN_PACKAGE_PRIORITY "extra")
set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://irods.org")
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS OFF)
set(CPACK_DEBIAN_PACKAGE_CONTROL_STRICT_PERMISSION ON)

set(CPACK_RPM_COMPONENT_INSTALL OFF)
set(CPACK_RPM_PACKAGE_RELEASE "1")
set(CPACK_RPM_PACKAGE_LICENSE "BSD-3-Clause")
set(CPACK_RPM_PACKAGE_VENDOR "iRODS Consortium")
set(CPACK_RPM_PACKAGE_URL "https://irods.org")
set(CPACK_RPM_PACKAGE_AUTOREQ 0)
set(CPACK_RPM_PACKAGE_AUTOPROV 0)

list(APPEND
  CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST_ADDITION
  "${CPACK_PACKAGING_INSTALL_PREFIX}etc/irods"
  "${CPACK_PACKAGING_INSTALL_PREFIX}var"
  "${CPACK_PACKAGING_INSTALL_PREFIX}var/lib"
  "${CPACK_PACKAGING_INSTALL_PREFIX}var/lib/irods"
  "${CPACK_PACKAGING_INSTALL_PREFIX}var/lib/irods/msiExecCmd_bin"
  )

set(CPACK_ARCHIVE_COMPONENT_INSTALL OFF)

set(CPACK_DEBIAN_PACKAGE_NAME "irods-hpc-data-to-compute-example")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "irods-server (= ${IRODS_CPACK_DEBIAN_PACKAGE_VERSION})")
set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_SOURCE_DIR}/postinst;")

set(CPACK_RPM_PACKAGE_NAME "irods-hpc-data-to-compute-example")
if (IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "centos" OR IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "centos linux")
  set(CPACK_RPM_PACKAGE_REQUIRES "irods-server = ${IRODS_VERSION}")
elseif (IRODS_LINUX_DISTRIBUTION_NAME STREQUAL "opensuse")
  set(CPACK_RPM_PACKAGE_REQUIRES "irods-server = ${IRODS_VERSION}")
endif()

set(CPACK_RPM_POST_INSTALL_SCRIPT_FILE "${CMAKE_SOURCE_DIR}/postinst")

if (NOT CPACK_GENERATOR)
  set(CPACK_GENERATOR ${IRODS_CPACK_GENERATOR} CACHE STRING "CPack generator to use, e.g. {DEB, RPM, TGZ}." FORCE)
  message(STATUS "Setting unspecified CPACK_GENERATOR to ${CPACK_GENERATOR}. This is the correct setting for normal builds.")
endif()

include(CPack)
