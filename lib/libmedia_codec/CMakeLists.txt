cmake_minimum_required(VERSION 3.5)
project (robomaster_media_codec)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Release)
SET(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/output)

find_path(AVCODEC_INCLUDE_DIR libavcodec/avcodec.h)
find_library(AVCODEC_LIBRARY avcodec)
find_path(AVUTIL_INCLUDE_DIR libavutil/avutil.h)
find_library(AVUTIL_LIBRARY avutil)
find_path(SWSCALE_INCLUDE_DIR libswscale/swscale.h)
find_library(SWSCALE_LIBRARY swscale)

find_path(OPUS_INCLUDE_DIRS
	NAMES opus/opus.h
	PATH_SUFFIXES include
)
find_library(OPUS_LIBRARIES NAMES opus)

include_directories(
	${AVCODEC_INCLUDE_DIR}
	${SWSCALE_INCLUDE_DIR}
	${AVUTIL_INCLUDE_DIR}
	${OPUS_INCLUDE_DIRS}
)

# set(PYBIND11_PYTHON_VERSION 3)
# add_subdirectory(pybind11)

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

IF (CMAKE_SYSTEM_NAME MATCHES "Windows")
	SET(OPUS_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/src/opus-share/include")
	SET(OPUS_LIBRARIES "${CMAKE_SOURCE_DIR}/src/opus-share/lib/opus.lib")
	SET(FFMEPG_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/src/ffmpeg-4.2.2-win64-dev/include")
	SET(FFMEPG_LIBRARIES_PATH "${CMAKE_SOURCE_DIR}/src/ffmpeg-4.2.2-win64-dev/lib")
	target_link_libraries(libmedia_codec PRIVATE
			${FFMEPG_LIBRARIES_PATH}/avcodec.lib
			${FFMEPG_LIBRARIES_PATH}/swscale.lib
			${FFMEPG_LIBRARIES_PATH}/avutil.lib
			${OPUS_LIBRARIES})
	target_include_directories(libmedia_codec PRIVATE
			${FFMEPG_INCLUDE_DIRS}
			${OPUS_INCLUDE_DIRS})
ELSE ()
	target_link_libraries(libmedia_codec PRIVATE
		${AVCODEC_LIBRARY}
		${SWSCALE_LIBRARY}
		${AVUTIL_LIBRARY}
		${OPUS_LIBRARY}
)
ENDIF (CMAKE_SYSTEM_NAME MATCHES "Windows")
