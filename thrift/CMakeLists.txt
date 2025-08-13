# Copyright (c) Meta Platforms, Inc. and affiliates.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

add_subdirectory(common)

if (THRIFT_COMPILER_ONLY OR build_all)
  add_subdirectory(compiler)
endif ()

if (THRIFT_LIB_ONLY OR build_all)
  add_subdirectory(annotation)
  add_subdirectory(lib)
endif ()

# Install CMake package configuration files for Thrift.
include(CMakePackageConfigHelpers)
configure_package_config_file(
  cmake/FBThriftConfig.cmake.in
  FBThriftConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_DIR}
  PATH_VARS
    CMAKE_INSTALL_DIR
    BIN_INSTALL_DIR
    INCLUDE_INSTALL_DIR
)
install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/FBThriftConfig.cmake
  DESTINATION ${CMAKE_INSTALL_DIR}
)
install(
  EXPORT fbthrift-exports
  FILE FBThriftTargets.cmake
  NAMESPACE FBThrift::
  DESTINATION ${CMAKE_INSTALL_DIR}
)

# Generate a pkg-config file so that downstream projects that don't use
# CMake can depend on fbthrift using pkg-config.
include(GenPkgConfig)

add_library(fbthrift_deps INTERFACE)
list(REMOVE_DUPLICATES FBTHRIFT_INCLUDE_DIRECTORIES)
target_include_directories(fbthrift_deps INTERFACE ${FBTHRIFT_INCLUDE_DIRECTORIES})
target_link_libraries(fbthrift_deps INTERFACE ${FBTHRIFT_LINK_LIBRARIES})

gen_pkgconfig_vars(FBTHRIFT_PKGCONFIG fbthrift_deps)
target_include_directories(fbthrift_deps
  BEFORE
  INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
)
target_include_directories(fbthrift_deps
  INTERFACE
    $<INSTALL_INTERFACE:include>
)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/pkgconfig/libfbthrift.pc.in
  ${CMAKE_CURRENT_BINARY_DIR}/libfbthrift.pc.gen
  @ONLY
)
file(
  GENERATE
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/libfbthrift.pc
  INPUT ${CMAKE_CURRENT_BINARY_DIR}/libfbthrift.pc.gen
  ${fbthrift_deps}
)
install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/libfbthrift.pc
  DESTINATION ${LIB_INSTALL_DIR}/pkgconfig
  COMPONENT dev
)

install(
  DIRECTORY annotation ${CMAKE_CURRENT_SOURCE_DIR}/annotation
  DESTINATION include/thrift
  FILES_MATCHING PATTERN *.thrift
)

install(
  DIRECTORY annotation ${CMAKE_CURRENT_BINARY_DIR}/annotation
  DESTINATION include/thrift
  FILES_MATCHING
    PATTERN *.h
    PATTERN *.tcc
    PATTERN CMakeFiles EXCLUDE
)
