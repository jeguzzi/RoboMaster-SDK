cmake_minimum_required(VERSION 3.15)
project (robomaster_media_codec)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_BUILD_TYPE Release)
set(VCPKG_BUILD_TYPE Release)

find_package(PkgConfig REQUIRED)
pkg_check_modules(AVCODEC    REQUIRED    IMPORTED_TARGET libavcodec)
pkg_check_modules(AVUTIL     REQUIRED    IMPORTED_TARGET libavutil)
pkg_check_modules(SWSCALE    REQUIRED    IMPORTED_TARGET libswscale)

find_package(Opus CONFIG REQUIRED)

set(Python3_FIND_STRATEGY LOCATION)
find_package(Python3 REQUIRED COMPONENTS Interpreter Development)
find_package(pybind11 3 QUIET)
if(NOT pybind11_FOUND)
  include(FetchContent)
  FetchContent_Declare(
    pybind11
    GIT_REPOSITORY https://github.com/pybind/pybind11.git
    GIT_TAG v3.0.1
    GIT_SHALLOW TRUE
    OVERRIDE_FIND_PACKAGE)
  set(PYBIND11_INSTALL OFF)
  set(INCLUDE_PYBIND11 false)
  FetchContent_MakeAvailable(pybind11)
  find_package(pybind11 REQUIRED)
endif()

pybind11_add_module(libmedia_codec src/media_codec.cpp)

target_link_libraries(libmedia_codec PRIVATE
	"-Wl,-Bsymbolic"
	PkgConfig::AVCODEC
	PkgConfig::AVUTIL
	PkgConfig::SWSCALE
	Opus::opus
)

set_target_properties(libmedia_codec PROPERTIES POSITION_INDEPENDENT_CODE TRUE)

include(CheckIPOSupported)
check_ipo_supported(RESULT ipo_supported OUTPUT error)
if(ipo_supported)
  set_property(TARGET libmedia_codec PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

install(
  TARGETS libmedia_codec
  DESTINATION .
  COMPONENT libmedia_codec)
