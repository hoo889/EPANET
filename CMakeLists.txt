# CMakeLists.txt - CMake configuration file for EPANET 2.0
#
# CMake is a cross-platform build tool. CMake generates platform native
# makefiles that can be used with your compiler of choice. CMake uses a
# generator concept to represent different build tooling. CMake automatically
# detects the platform it is running on and generates the appropriate makefiles
# for the platform default compiler. Different generators can also be specified.
#
# Note: CMake requires that your platform build system and compiler are
# properly installed. Build using Visual Studio requires msbuild shell.
#
# Example Usage:
#   mkdir buildproducts
#   cd buildproducts
#   cmake ..
#   make
#
# Building MSYS on Windows:
#   ...
#   cmake -G "MSYS Makefiles" ..
#   make
#
# Building Visual Studio on Windows:
#   ...
#   cmake -G "Visual Studio 10 2010" ..
#   msbuild /p:Configuration=Release ALL_BUILD.vcxproj
#
# Generic Invocation:
#   cmake -E make_directory buildprod
#   cd build
#   cmake -G GENERATOR -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=1 ..
#   cmake --build . --target SOME_TARGET --config Release
#
# More information:
#   cmake --help
#
#   CMake is available at https://cmake.org/download/
#

cmake_minimum_required (VERSION 2.8.8)

project(EPANET)
add_subdirectory(tools/epanet-output)

IF (BUILD_TESTS)
  add_subdirectory(tests)
ENDIF (BUILD_TESTS)

# Sets for output directory for executables and libraries.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Sets the position independent code property for all targets
SET(CMAKE_POSITION_INDEPENDENT_CODE ON)


IF (APPLE)
SET(INSTALL_NAME_DIR @executable_path/../lib)
SET(CMAKE_MACOSX_RPATH 1)
ENDIF (APPLE)

IF (MSVC)
  set(CMAKE_C_FLAGS_RELEASE "/GL")
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
ENDIF (MSVC)


#include_directories(include src)


# configure file groups
file(GLOB EPANET_SOURCES src/*.c)
#set(EPANET_API_HEADER include/epanet2.h)
set(EPANET_CLI_SOURCES run/main.c)

file(GLOB EPANET_LIB_ALL src/*)
source_group("Library" FILES ${EPANET_LIB_ALL})
source_group("CLI" REGULAR_EXPRESSION "run/.*")


# the shared library
add_library(epanet SHARED ${EPANET_SOURCES})  #${EPANET_API_HEADER})
target_include_directories(epanet PUBLIC ${PROJECT_SOURCE_DIR}/include)

# create export lib so we can link against dll using Visual Studio
#include(GenerateExportHeader)
#GENERATE_EXPORT_HEADER(epanet
#    BASE_NAME epanet
#    EXPORT_MACRO_NAME DLLEXPORT
#    EXPORT_FILE_NAME epanet_export.h
#    STATIC_DEFINE SHARED_EXPORTS_BUILT_AS_STATIC)
#
#file(COPY ${CMAKE_CURRENT_BINARY_DIR}/epanet_export.h
#    DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/include)


# the standalone executable
add_executable(runepanet ${EPANET_CLI_SOURCES})

if(NOT MSVC)
    target_link_libraries(runepanet LINK_PUBLIC epanet m)
else(NOT MSVC)
	target_link_libraries(runepanet LINK_PUBLIC epanet)
endif(NOT MSVC)
