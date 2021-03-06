# Copyright 2017 Proyectos y Sistemas de Mantenimiento SL (eProsima).
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

###############################################################################
# CMake build rules for ShapesDemo
###############################################################################
cmake_minimum_required(VERSION 2.8.12)


file(READ version.pri SHAPESVERSION)


set(IS_TOP_LEVEL TRUE)
if(PROJECT_SOURCE_DIR)
    set(IS_TOP_LEVEL FALSE)
endif()

# Set CMAKE_BUILD_TYPE to Release by default.
if(DEFINED CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        )
else()
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        )
endif()

project("ShapesDemo")
set(PROJECT_NAME_STYLED "ShapesDemo")
set(PROJECT_NAME_LARGE "Shapes Demo")
string(TOUPPER "${PROJECT_NAME}" PROJECT_NAME_UPPER)
set(${PROJECT_NAME}_DESCRIPTION_SUMMARY "Shapes Demo for eProsima Fast RTPS")
set(${PROJECT_NAME}_DESCRIPTION "eProsima ${PROJECT_NAME_LARGE} library provides publication/subscription communication using RTPS protocol.")

message(STATUS "Configuring ${PROJECT_NAME_LAGE}")

###############################################################################
# Warning level
###############################################################################
if(MSVC OR MSVC_IDE)
    if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
    endif()
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wextra -Wno-unknown-pragmas")
endif()

###############################################################################
# GCC colors if using CCache
###############################################################################
if("${CMAKE_CXX_COMPILER_LAUNCHER}" STREQUAL "ccache" AND
        CMAKE_COMPILER_IS_GNUCXX AND
        CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 4,8)
    add_compile_options(-fdiagnostics-color=always)
endif()

###############################################################################
# Test system configuration
###############################################################################
include(${PROJECT_SOURCE_DIR}/cmake/common/check_configuration.cmake)
check_stdcxx()
check_compile_feature()
check_endianness()

###############################################################################
# Check MSVC architecture
###############################################################################
if(MSVC OR MSVC_IDE)
    check_msvc_arch()
endif()

###############################################################################
# Load CMake modules
###############################################################################
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake/modules)

###############################################################################
# Load external eProsima projects.
###############################################################################
###
# Installation directories
###
set(BIN_INSTALL_DIR bin/ CACHE PATH "Installation directory for binaries")
set(INCLUDE_INSTALL_DIR include/ CACHE PATH "Installation directory for C++ Headers")
set(LIB_INSTALL_DIR lib/ CACHE PATH "Installation directory for libraries")
set(DATA_INSTALL_DIR share/ CACHE PATH "Installation directory for data")
if(WIN32)
    set(DOC_DIR "doc")
else()
    set(DOC_DIR "${DATA_INSTALL_DIR}/doc")
endif()
set(DOC_INSTALL_DIR ${DOC_DIR} CACHE PATH "Installation directory for documentation")
if(WIN32)
    set(LICENSE_INSTALL_DIR . CACHE PATH "Installation directory for licenses")
else()
    set(LICENSE_INSTALL_DIR ${DATA_INSTALL_DIR}/${PROJECT_NAME} CACHE PATH "Installation directory for licenses")
endif()

set(CMAKE_BUILD_TYPE Release CACHE STRING "Build Type")

###############################################################################
# eProsima build options
###############################################################################
option(EPROSIMA_BUILD "Activate internal building" OFF)

###############################################################################
# Load external eProsima projects.
###############################################################################
include(${PROJECT_SOURCE_DIR}/cmake/common/eprosima_libraries.cmake)
option(THIRDPARTY "Activate the build of thirdparties" OFF)

if(EPROSIMA_BUILD)
    set(THIRDPARTY ON)
endif()

eprosima_find_package(fastcdr)
eprosima_find_package(fastrtps OPTION TINYXML2_FROM_SOURCE)

###############################################################################
# Compile
###############################################################################
find_package(Qt5Core 5.9 REQUIRED)
find_package(Qt5Gui REQUIRED)
find_package(Qt5Widgets REQUIRED)

set( MOC_HEADERS
    include/eprosimashapesdemo/qt/DrawArea.h
    include/eprosimashapesdemo/qt/UpdateThread.h
    include/eprosimashapesdemo/qt/mainwindow.h
    include/eprosimashapesdemo/qt/optionsdialog.h
    include/eprosimashapesdemo/qt/publishdialog.h
    include/eprosimashapesdemo/qt/subscribedialog.h
    )

set( UIS
    forms/mainwindow.ui
    forms/optionsdialog.ui
    forms/publishdialog.ui
    forms/subscribedialog.ui
    )
    
set( RESOURCES
    images/eprosimalogo.qrc
    )


QT5_WRAP_UI( UI_HEADERS ${UIS} )
QT5_WRAP_CPP( MOC_SRCS ${MOC_HEADERS} )
QT5_ADD_RESOURCES( RSCS ${RESOURCES} )

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOUIC_SEARCH_PATHS forms)

message(STATUS "Configuring ShapesDemo...")
set( SHAPESDEMO_SOURCES
    src/qt/DrawArea.cpp
    src/qt/mainwindow.cpp
    src/qt/publishdialog.cpp
    src/qt/subscribedialog.cpp
    src/shapesdemo/Shape.cxx
    src/shapesdemo/ShapesDemo.cpp
    src/shapesdemo/ShapePubSubTypes.cxx
    src/main.cpp
    src/shapesdemo/ShapePublisher.cpp
    src/qt/UpdateThread.cpp
    src/shapesdemo/ShapeSubscriber.cpp
    src/qt/optionsdialog.cpp
    src/qt/ContentFilterSelector.cpp
    src/shapesdemo/ShapeHistory.cpp
    images/eprosima_icon.rc
    )

if(THIRDPARTY)
    set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR}")
    set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
endif()

if(WIN32)
    add_executable(${PROJECT_NAME} WIN32 ${SHAPESDEMO_SOURCES} ${MOC_SRCS} ${UI_HEADERS} ${RSCS} )
else()
    add_executable(${PROJECT_NAME} ${SHAPESDEMO_SOURCES} ${MOC_SRCS} ${UI_HEADERS} ${RSCS} )
endif()

target_compile_definitions(${PROJECT_NAME} PRIVATE -D${SHAPESVERSION})
target_include_directories(${PROJECT_NAME} PRIVATE include ${IU_HEADERS})
target_link_libraries(${PROJECT_NAME} fastrtps fastcdr Qt5::Widgets Qt5::Core Qt5::Gui)

###############################################################################
# Install
###############################################################################
if(THIRDPARTY)
    if(NOT WIN32)
        # Install libraries
        install(DIRECTORY ${PROJECT_BINARY_DIR}/external/install/${LIB_INSTALL_DIR}/
            DESTINATION ${LIB_INSTALL_DIR}
            USE_SOURCE_PERMISSIONS
            COMPONENT libraries
            )
    endif()

    # Install licenses
    install(DIRECTORY ${PROJECT_BINARY_DIR}/external/install/licenses/
        DESTINATION ${LICENSE_INSTALL_DIR}
        COMPONENT licenses
        OPTIONAL
        )
endif()

install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION ${BIN_INSTALL_DIR}
    LIBRARY DESTINATION ${BIN_INSTALL_DIR}
    ARCHIVE DESTINATION ${BIN_INSTALL_DIR}
    COMPONENT binaries
    )

# Install dependecies
include(${PROJECT_SOURCE_DIR}/cmake/dev/install_prerequisites.cmake)
install_prerequisites(${PROJECT_NAME})
