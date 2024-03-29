cmake_minimum_required(VERSION 3.16)
project(AppBase)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules")

include(InstallDirectoryPermissions)
include(GNUInstallDirs)

file(GLOB HEADERS "include/appbase/*.hpp")

set(CMAKE_EXPORT_COMPILE_COMMANDS "ON")

if(CMAKE_CXX_STANDARD EQUAL 98 OR CMAKE_CXX_STANDARD LESS 17)
  message(FATAL_ERROR "appbase requires c++17 or newer")
elseif(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)
find_package(Threads)

find_package(Boost 1.60 REQUIRED)

if(APPLE)
  # Apple Specific Options Here
  message(STATUS "Configuring AppBase on OS X")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-conversion -Wno-deprecated-declarations")
else(APPLE)
  # Linux Specific Options Here
  message(STATUS "Configuring AppBase on Linux")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
  if(FULL_STATIC_BUILD)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static-libstdc++ -static-libgcc")
  endif(FULL_STATIC_BUILD)
endif(APPLE)


if(ENABLE_COVERAGE_TESTING)
  set(CMAKE_CXX_FLAGS "--coverage ${CMAKE_CXX_FLAGS}")
endif()

add_library(appbase application.cpp)
target_link_libraries(appbase Threads::Threads)
target_include_directories(appbase
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${Boost_INCLUDE_DIR})

set_target_properties(appbase PROPERTIES PUBLIC_HEADER "${HEADERS}")

set(CPACK_PACKAGING_INSTALL_PREFIX /)

install(TARGETS appbase
  RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_FULL_INCLUDEDIR}/appbase)
install_directory_permissions(DIRECTORY ${CMAKE_INSTALL_FULL_INCLUDEDIR}/appbase)

option(BUILD_EXAMPLES "Build an appbase application example" OFF)

if(BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()
