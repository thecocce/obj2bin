 #*************************************************************************
 #   CMakeLists.txt  --  This file is part of obj2bit.                    *
 #                                                                        *
 #   Copyright (C) 2016, Ivo Filot                                        *
 #                                                                        *
 #   obj2bit is free software: you can redistribute it and/or modify      *
 #   it under the terms of the GNU General Public License as published    *
 #   by the Free Software Foundation, either version 3 of the License,    *
 #   or (at your option) any later version.                               *
 #                                                                        *
 #   obj2bit is distributed in the hope that it will be useful,           *
 #   but WITHOUT ANY WARRANTY; without even the implied warranty          *
 #   of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.              *
 #   See the GNU General Public License for more details.                 *
 #                                                                        *
 #   You should have received a copy of the GNU General Public License    *
 #   along with this program.  If not, see http://www.gnu.org/licenses/.  *
 #                                                                        *
 #*************************************************************************/

# set minimum cmake requirements
cmake_minimum_required(VERSION 2.8)
project (obj2bit)

# add custom directory to look for .cmake files
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake/modules )

# Enable release build
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# add OS specific
if(APPLE)
    add_definitions(-D_APPLE)
    add_definitions(-DEIGEN_NO_DEBUG)
    SET(BOOST_INCLUDEDIR "/opt/local/include")
    SET(BOOST_LIBRARYDIR "/opt/local/lib")
else()
    add_definitions(-DEIGEN_NO_DEBUG -Wno-literal-suffix)
    SET(BOOST_INCLUDEDIR "/usr/include")
    SET(BOOST_LIBRARYDIR "/usr/lib/x86_64-linux-gnu")
endif()

# set Boost
set (Boost_NO_SYSTEM_PATHS ON)
set (Boost_USE_MULTITHREADED ON)
set (Boost_USE_STATIC_LIBS ON)
set (Boost_USE_STATIC_RUNTIME OFF)
set (BOOST_ALL_DYN_LINK OFF)

# Include libraries
find_package(PkgConfig REQUIRED)
find_package(BZip2 REQUIRED)
find_package(Boost COMPONENTS chrono regex iostreams system serialization filesystem log thread REQUIRED)
pkg_check_modules(TCLAP tclap REQUIRED)

# Set include folders
include_directories(${CMAKE_CURRENT_SOURCE_DIR}
                    ${CMAKE_BINARY_DIR}
                    ${Boost_INCLUDE_DIRS})

# Add sources
file(GLOB_RECURSE SOURCES "*.cpp")
add_executable(obj2bit ${SOURCES})

# Set C++14
add_definitions(-std=c++14)

# Link libraries
if(UNIX AND NOT APPLE)
    SET(CMAKE_EXE_LINKER_FLAGS "-Wl,-rpath=\$ORIGIN/lib")
endif()
if(APPLE)
    SET(CMAKE_MACOSX_RPATH TRUE)
    SET_TARGET_PROPERTIES(obj2bit PROPERTIES INSTALL_RPATH "@executable_path/lib")
    SET(CMAKE_EXE_LINKER_FLAGS "-L${GLEW_LIBRARY_DIRS}")
    target_link_libraries(obj2bit ${Boost_LIBRARIES})
else()
    target_link_libraries(obj2bit ${Boost_LIBRARIES} ${BZIP2_LIBRARIES})
endif()

# add Boost definition
add_definitions(-DBOOST_LOG_DYN_LINK)

# add Wno-literal-suffix to suppress warning messages
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS}")

###
# Installing
##
install (TARGETS obj2bit DESTINATION bin)
