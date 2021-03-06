cmake_minimum_required(VERSION 3.8)
include(${CMAKE_TOOLCHAIN_FILE})

option(BUILD_TESTS "Configure CMake to build tests (or not)" ON)

project(jwt-cpp)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(OpenSSL 1.1 REQUIRED)

add_library(jwt-cpp INTERFACE)
add_library(jwt-cpp::jwt-cpp ALIAS jwt-cpp)

target_compile_features(jwt-cpp INTERFACE cxx_std_11)

set(JWT_INCLUDE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(
  jwt-cpp INTERFACE $<BUILD_INTERFACE:${JWT_INCLUDE_PATH}>
                    $<INSTALL_INTERFACE:include>)

target_link_libraries(jwt-cpp INTERFACE OpenSSL::SSL OpenSSL::Crypto)

set(JWT_HEADERS ${JWT_INCLUDE_PATH}/jwt-cpp/base.h
                ${JWT_INCLUDE_PATH}/jwt-cpp/jwt.h)
set(PICO_HEADER ${JWT_INCLUDE_PATH}/picojson/picojson.h)

include(CMakePackageConfigHelpers)
set(INCLUDE_INSTALL_DIR include)

configure_package_config_file(
  ${CMAKE_CURRENT_LIST_DIR}/cmake/jwt-cpp-config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/jwt-cpp-config.cmake INSTALL_DESTINATION
  ${CMAKE_INSTALL_PREFIX}/jwt-cpp/cmake PATH_VARS INCLUDE_INSTALL_DIR)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/jwt-cpp-config-version.cmake
  VERSION 0.3.1 # TBA
  COMPATIBILITY ExactVersion)

install(FILES ${JWT_HEADERS} DESTINATION include/jwt-cpp)
install(FILES ${PICO_HEADER} DESTINATION include/picojson)
install(
  TARGETS jwt-cpp
  EXPORT jwt-cpp
  DESTINATION jwt-cpp)
install(
  EXPORT jwt-cpp
  NAMESPACE jwt-cpp::
  DESTINATION jwt-cpp)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/jwt-cpp-config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/jwt-cpp-config-version.cmake
        DESTINATION ${CMAKE_INSTALL_PREFIX}/jwt-cpp)

if(BUILD_TESTS)
  add_subdirectory(tests)
endif()
