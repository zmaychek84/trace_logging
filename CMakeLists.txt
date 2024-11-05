# Copyright © 2023 Advanced Micro Devices, Inc. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http:#www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or Implied.
# See the License for the specific language governing permissions and
# limitations under the License.
cmake_minimum_required(VERSION 3.12)

project(trace-logging VERSION 1.0.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(XNAMESPACE "vart")

add_library(${PROJECT_NAME} include/vitis/ai/tracelogging.hpp
                              src/tracelogging.cpp)

add_library(${XNAMESPACE}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

set_target_properties(
  ${PROJECT_NAME}
  PROPERTIES
  VERSION "${PROJECT_VERSION}"
  SOVERSION "${PROJECT_VERSION_MAJOR}"
  LIBRARY_OUTPUT_NAME ${XNAMESPACE}-${PROJECT_NAME}
  OUTPUT_NAME ${XNAMESPACE}-${PROJECT_NAME}
  CXX_STANDARD 17
  )

option(VART_TL_ENABLE "Enable VART Trace Logging module." ON)
option(BUILD_TEST "Build Tests." OFF)

target_compile_definitions(
  ${PROJECT_NAME}
  PUBLIC TL_ENABLE_BUILD_FLAG=$<BOOL:${VART_TL_ENABLE}> _CRT_SECURE_NO_WARNINGS
  )

target_include_directories(
  ${PROJECT_NAME}
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
         $<INSTALL_INTERFACE:include>
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src
  )

if(CMAKE_SOURCE_DIR STREQUAL trace-logging_SOURCE_DIR)
install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}-targets
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib)

install(
  EXPORT ${PROJECT_NAME}-targets
  NAMESPACE ${XNAMESPACE}::
  DESTINATION share/cmake/${PROJECT_NAME}
  )

install(
  DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/vitis/ai/
  DESTINATION include/vitis/ai
  FILES_MATCHING
  PATTERN "*.hpp"
  PATTERN "*.h"
  )
endif()

if(BUILD_TEST)
  add_executable(test_tracelogging test/test_tracelogging.cpp)
  target_link_libraries(test_tracelogging ${PROJECT_NAME})
endif()

include(CMakePackageConfigHelpers)

file(RELATIVE_PATH REL_INCLUDE_DIR
  ${CMAKE_INSTALL_PREFIX}/share/cmake/${PROJECT_NAME}
  ${CMAKE_INSTALL_PREFIX}/include
)

set(CONF_INCLUDE_DIRS "\${${PROJECT_NAME}_CMAKE_DIR}/${REL_INCLUDE_DIR}")

configure_file(
  ${PROJECT_SOURCE_DIR}/cmake/config.in
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}-config.cmake
  @ONLY
  )

write_basic_package_version_file (
  ${CMAKE_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
  )

if(CMAKE_SOURCE_DIR STREQUAL trace-logging_SOURCE_DIR)
 install(
  FILES ${CMAKE_BINARY_DIR}/${PROJECT_NAME}-config.cmake ${CMAKE_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  COMPONENT runtime
  DESTINATION share/cmake/${PROJECT_NAME}
  )
endif()
