cmake_minimum_required( VERSION 2.8 FATAL_ERROR )
project (urdfdom CXX C)

set (URDF_MAJOR_VERSION 0)
set (URDF_MINOR_VERSION 2)
set (URDF_PATCH_VERSION 8)

set (URDF_VERSION ${URDF_MAJOR_VERSION}.${URDF_MINOR_VERSION}.${URDF_PATCH_VERSION})

message (STATUS "${PROJECT_NAME} version ${URDF_VERSION}")

include(GNUInstallDirs)

# set the default build type
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# If compiler support symbol visibility, enable it.
include(CheckCCompilerFlag)
check_c_compiler_flag(-fvisibility=hidden HAS_VISIBILITY)
if (HAS_VISIBILITY)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
endif()

# This shouldn't be necessary, but there has been trouble
# with MSVC being set off, but MSVCXX ON.
if(MSVC OR MSVC90 OR MSVC10)
  set(MSVC ON)
endif (MSVC OR MSVC90 OR MSVC10)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(SearchForTinyXML)
include_directories(${tinyxml_include_dirs})
link_directories(${tinyxml_library_dirs})
add_definitions(${tinyxml_cflags})

find_package(urdfdom_headers REQUIRED)
include_directories(SYSTEM ${urdfdom_headers_INCLUDE_DIRS})

find_package(console_bridge REQUIRED)
include_directories(SYSTEM ${console_bridge_INCLUDE_DIRS})
link_directories(${console_bridge_LIBRARY_DIRS})

find_package(Boost REQUIRED system thread)
include_directories(${Boost_INCLUDE_DIR})
link_directories(${Boost_LIBRARY_DIRS})

add_subdirectory(urdf_parser)

set(PKG_NAME ${PROJECT_NAME})
set(PKG_LIBRARIES urdfdom_sensor urdfdom_model_state urdfdom_model urdfdom_world)
set(PKG_DEPENDS urdfdom_headers console_bridge)
set(cmake_conf_file "${CMAKE_CURRENT_SOURCE_DIR}/cmake/urdfdom-config.cmake")
configure_file("${cmake_conf_file}.in" "${CMAKE_BINARY_DIR}/${cmake_conf_file}" @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/${cmake_conf_file}
  DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake/ COMPONENT cmake)

# Make the package config file
if (NOT MSVC)
  set(PKG_DESC "Unified Robot Description Format")
  set(PKG_DEPENDS "urdfdom_headers console_bridge") # make the list separated by spaces instead of ;
  set(PKG_URDF_LIBS "-lurdfdom_sensor -lurdfdom_model_state -lurdfdom_model -lurdfdom_world")
  set(pkg_conf_file "${CMAKE_CURRENT_SOURCE_DIR}/cmake/pkgconfig/urdfdom.pc")
  configure_file("${pkg_conf_file}.in" "${CMAKE_BINARY_DIR}/${pkg_conf_file}" @ONLY)
  install(FILES ${CMAKE_BINARY_DIR}/${pkg_conf_file}
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig/ COMPONENT pkgconfig)
endif()


message(STATUS "Configuration successful. Type make to compile urdfdom")

SET_DIRECTORY_PROPERTIES(PROPERTIES
  ADDITIONAL_MAKE_CLEAN_FILES ${CMAKE_BINARY_DIR}/cmake/urdfdom-config.cmake
  ADDITIONAL_MAKE_CLEAN_FILES ${CMAKE_BINARY_DIR}/cmake/pkgconfig/urdfdom.pc)
