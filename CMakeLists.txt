# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.20.0)
find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(external_lib)

target_sources(app PRIVATE src/main.c)

# The external static library that we are linking with does not know
# how to build for this platform so we export all the flags used in
# this zephyr build to the external build system.
#
# Other external build systems may be self-contained enough that they
# do not need any build information from zephyr. Or they may be
# incompatible with certain zephyr options and need them to be
# filtered out.
zephyr_get_include_directories_for_lang_as_string(       C includes)
zephyr_get_system_include_directories_for_lang_as_string(C system_includes)
zephyr_get_compile_definitions_for_lang_as_string(       C definitions)
zephyr_get_compile_options_for_lang_as_string(           C options)

set(external_project_cflags
  "${includes} ${definitions} ${options} ${system_includes}"
  )

include(ExternalProject)

# Add an external project to be able download and build the third
# party library. In this case downloading is not necessary as it has
# been committed to the repository.
set(sqlitelib_src_dir   ${CMAKE_CURRENT_SOURCE_DIR}/sqlitelib)
set(sqlitelib_build_dir ${CMAKE_CURRENT_BINARY_DIR}/sqlitelib)

set(SQLITELIB_LIB_DIR     ${sqlitelib_build_dir}/lib)
set(SQLITELIB_INCLUDE_DIR ${sqlitelib_src_dir}/include)

if(CMAKE_GENERATOR STREQUAL "Unix Makefiles")
# https://www.gnu.org/software/make/manual/html_node/MAKE-Variable.html
set(submake "$(MAKE)")
else() # Obviously no MAKEFLAGS. Let's hope a "make" can be found somewhere.
set(submake "make")
endif()

ExternalProject_Add(
  sqlite3_module                 # Name for custom target
  PREFIX     ${sqlitelib_build_dir} # Root dir for entire project
  SOURCE_DIR ${sqlitelib_src_dir}
  BINARY_DIR ${sqlitelib_src_dir} # This particular build system is invoked from the root
  CONFIGURE_COMMAND ""    # Skip configuring the project, e.g. with autoconf
  BUILD_COMMAND
  ${submake}
  PREFIX=${sqlitelib_build_dir}
  CC=${CMAKE_C_COMPILER}
  AR=${CMAKE_AR}
  CFLAGS=${external_project_cflags}
  INSTALL_COMMAND ""      # This particular build system has no install command
  BUILD_BYPRODUCTS ${SQLITELIB_LIB_DIR}/libsqlite3.a
  DEPENDS zephyr_interface
  )

# Create a wrapper CMake library that our app can link with
add_library(sqlitelib_lib STATIC IMPORTED GLOBAL)
add_dependencies(
  sqlitelib_lib
  sqlitelib_project
  )
set_target_properties(sqlitelib_lib PROPERTIES IMPORTED_LOCATION             ${SQLITELIB_LIB_DIR}/libsqlite3.a)
set_target_properties(sqlitelib_lib PROPERTIES INTERFACE_INCLUDE_DIRECTORIES ${SQLITELIB_INCLUDE_DIR})

target_link_libraries(app PUBLIC sqlitelib_lib)