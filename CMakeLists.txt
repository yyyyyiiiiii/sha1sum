cmake_minimum_required(VERSION 3.28.0)

project(shasum LANGUAGES CXX DESCRIPTION "Print or check SHA checksums." VERSION 0.0.0)

option(BUILD_TEST "Build the test" OFF)
option(CMAKE_BUILD_TYPE "Build type: [Debug, Release]" Release)

if(CMAKE_BUILD_TYPE STREQUAL Debug)
  set(FETCHCONTENT_QUIET FALSE)
endif()

add_compile_options(
  $<$<CONFIG:Release>:$<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:GNU>>:-O3>>
  $<$<CONFIG:Release>:$<$<CXX_COMPILER_ID:MSVC>:/O2>>
  $<$<CONFIG:Debug>:$<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:GNU>>:-g>>
  $<$<CONFIG:Debug>:$<$<CXX_COMPILER_ID:MSVC>:/DEBUG>>
)

find_package(cxxopts)
if (NOT cxxopts_FOUND)
  include(FetchContent)

  FetchContent_Declare(
    cxxopts
    GIT_REPOSITORY https://github.com/jarro2783/cxxopts.git
    GIT_TAG master
  )

  FetchContent_MakeAvailable(cxxopts)
endif()

set(PROJECT_ccm_PREFIX ".project_ccm_prefix")
set(PROJECT_ccm_DIR "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_ccm_PREFIX}")
set(PROJECT_ccm_PATH "${PROJECT_ccm_DIR}/project.ccm")
configure_file(project.ccm.in "${PROJECT_ccm_PATH}" @ONLY)

add_executable(${PROJECT_NAME})
target_sources(
  ${PROJECT_NAME}
  PRIVATE main.cc
  PUBLIC FILE_SET CXX_MODULES FILES
  app.ccm
  options.ccm
  hash.ccm
  istream.ccm
  "${PROJECT_ccm_PATH}"
)
target_link_libraries(${PROJECT_NAME} PRIVATE cxxopts::cxxopts)
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_23)
target_include_directories(${PROJECT_NAME} PUBLIC "${PROJECT_HH_INCLUDE_DIR}")

if (BUILD_TEST)
  add_subdirectory(test)
endif()

install(TARGETS ${PROJECT_NAME} COMPONENT ${PROJECT_NAME})
