#
# This file is part of the Sound library.
# Copyright (C) 2010-2013 Benjamin Eikel <benjamin@eikel.org>
#
# This library is subject to the terms of the Mozilla Public License, v. 2.0.
# You should have received a copy of the MPL along with this library; see the 
# file LICENSE. If not, you can obtain one at http://mozilla.org/MPL/2.0/.
#
cmake_minimum_required(VERSION 2.8.11)
project(Sound CXX)

set(SOUND_VERSION_MAJOR 0)
set(SOUND_VERSION_MINOR 2)
set(SOUND_VERSION_PATCH 0)
set(SOUND_VERSION_STRING ${SOUND_VERSION_MAJOR}.${SOUND_VERSION_MINOR}.${SOUND_VERSION_PATCH})

# Set up install directories
include(GNUInstallDirs)
set(CMAKE_INSTALL_CMAKECONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/Sound)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

add_library(Sound
	Buffer.cpp
	Listener.cpp
	Sound.cpp
	Source.cpp
)

# Dependency to OpenAL
find_package(OpenAL REQUIRED)
target_include_directories(Sound PRIVATE ${OPENAL_INCLUDE_DIR})
target_link_libraries(Sound LINK_PRIVATE ${OPENAL_LIBRARY})

# Dependency to SDL2
set(SDL2_BUILDING_LIBRARY ON)
find_package(SDL2 2.0.0 REQUIRED)
target_include_directories(Sound PRIVATE ${SDL2_INCLUDE_DIRS})
target_link_libraries(Sound LINK_PRIVATE ${SDL2_LIBRARIES})

# Dependency to Util
if(NOT TARGET Util)
	find_package(Util 0.2.0 REQUIRED NO_MODULE)
endif()
target_link_libraries(Sound LINK_PUBLIC Util)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
	set_property(TARGET Sound APPEND_STRING PROPERTY COMPILE_FLAGS "-std=c++11 ")
elseif(COMPILER_SUPPORTS_CXX0X)
	set_property(TARGET Sound APPEND_STRING PROPERTY COMPILE_FLAGS "-std=c++0x ")
else()
	message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# Set version of library
set_target_properties(Sound PROPERTIES VERSION ${SOUND_VERSION_STRING}
                                       SOVERSION ${SOUND_VERSION_MAJOR})

# Install the header files
file(GLOB SOUND_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/*.h")
install(FILES ${SOUND_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/Sound COMPONENT headers)

# Install the library
install(TARGETS Sound EXPORT LibraryExport
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtimelibraries
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtimelibraries
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT developmentlibraries
)

# Set include directories for users of this library
target_include_directories(Sound
                           INTERFACE
                           "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>"
                           "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")

# Library export
install(EXPORT LibraryExport DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR} FILE SoundTargets.cmake COMPONENT developmentlibraries)

# Create the configuration files
include(CMakePackageConfigHelpers)
configure_package_config_file(SoundConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/SoundConfig.cmake
                              INSTALL_DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR}
                              PATH_VARS CMAKE_INSTALL_CMAKECONFIGDIR
                              NO_SET_AND_CHECK_MACRO
                              NO_CHECK_REQUIRED_COMPONENTS_MACRO)
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/SoundConfigVersion.cmake
                                 VERSION ${SOUND_VERSION_STRING}
                                 COMPATIBILITY SameMajorVersion)

# Install the configuration files
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/SoundConfig.cmake ${CMAKE_CURRENT_BINARY_DIR}/SoundConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_CMAKECONFIGDIR} COMPONENT developmentlibraries)

# Packaging
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_NAME "libsound${SOUND_VERSION_MAJOR}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Sound library")
set(CPACK_PACKAGE_VENDOR "Benjamin Eikel; Claudius Jaehn; Ralf Petring")
set(CPACK_PACKAGE_CONTACT "Benjamin Eikel <benjamin@eikel.org>")
set(CPACK_PACKAGE_VERSION_MAJOR ${SOUND_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${SOUND_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${SOUND_VERSION_PATCH})
set(CPACK_PACKAGE_FILE_NAME "libSound")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_STRIP_FILES ON)

include(CPack)
