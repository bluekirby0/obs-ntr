cmake_minimum_required(VERSION 2.8.12)

project(obs-ntr)

# Standalone build
if(NOT OBS_OUTPUT_DIR)
	set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
	include(ObsHelpers)

	set(CMAKE_C_STANDARD 11)
	set(STANDALONE_PLUGIN TRUE)

	find_path(OBS_INCLUDE_DIRS
		obs-module.h
		HINTS $ENV{OBS_DIR} ${OBS_DIR}
		PATH_SUFFIXES
			libobs)
	find_library(OBS_LIBRARIES
		NAMES obs libobs
		HINTS $ENV{OBS_DIR} ${OBS_DIR}
		PATH_SUFFIXES
			build_rel/libobs
			libobs)
# Build as submodule of obs-studio
else()
	set(OBS_INCLUDE_DIRS
		../../libobs)
	set(OBS_LIBRARIES
		libobs)
endif()

find_package(TurboJPEG REQUIRED)
find_package (Threads REQUIRED)

include_directories(
	${OBS_INCLUDE_DIRS}
	${TurboJPEG_INCLUDE_DIRS})

#link_directories(
#	${TurboJPEG_LIBRARIES})

set(obs-ntr_SOURCES
	src/obs-ntr.c)

add_library(obs-ntr MODULE
	${obs-ntr_SOURCES})
target_link_libraries(obs-ntr
	${TurboJPEG_LIBRARIES}
	${OBS_LIBRARIES}
	${CMAKE_THREAD_LIBS_INIT})

if(STANDALONE_PLUGIN)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		set(_bit_suffix "64bit/")
	else()
		set(_bit_suffix "32bit/")
	endif()
	if(APPLE)
		install(TARGETS obs-ntr
			LIBRARY DESTINATION "Library/Application Support/obs-studio/plugins/libobs-ntr/bin/")
		install(DIRECTORY data
			DESTINATION "Library/Application Support/obs-studio/plugins/libobs-ntr/data")
		# for packaging, copy all dependencies, change dependency links
		# run pkgbuild --identifier user.obsproject.kurufu.obs-ntr --root ./obs-obs-ntr-install/ obs-ntr-mac.pkg
	else() # Not apple so normal layout
		install(TARGETS obs-ntr
			LIBRARY DESTINATION "obs-plugins/${_bit_suffix}/")
		install(DIRECTORY data
			DESTINATION data/obs-plugins/obs-ntr)
	endif()
else()
	install_obs_plugin_with_data(obs-ntr data)
endif()

