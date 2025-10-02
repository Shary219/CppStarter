cmake_minimum_required(VERSION 3.16...4.1.1 FATAL_ERROR)

#--------------------------------------------------------------------
# Project and version.
#--------------------------------------------------------------------

set (CPPSTARTER_VERSION_MAJOR 0)
set (CPPSTARTER_VERSION_MINOR 1)
set (CPPSTARTER_VERSION_PATCH 1)
set (CPPSTARTER_VERSION_STRING ${CPPSTARTER_VERSION_MAJOR}.${CPPSTARTER_VERSION_MINOR}.${CPPSTARTER_VERSION_PATCH})

project(Starter
        VERSION ${CPPSTARTER_VERSION_STRING} 
        DESCRIPTION "Starter Project for C/C++"
        HOMEPAGE_URL "https://github.com/djoezeke/CppStarter"
        LANGUAGES  C
)

#--------------------------------------------------------------------
# Preparations depending on the specified build options  
#--------------------------------------------------------------------

# Check if CPPSTARTER is the main project
if (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(CPPSTARTER_IS_TOP_LEVEL ON)
else()
  set(CPPSTARTER_IS_TOP_LEVEL OFF)
endif()

#--------------------------------------------------------------------
# Configure variables for the library   
#--------------------------------------------------------------------
include(GNUInstallDirs)

set(CPPSTARTER_LIB_NAME "Starter" CACHE STRING "Base name of library output name")

set(CPPSTARTER_SOURCES starter.c)
set(CPPSTARTER_INCLUDES starter.h)
set(CPPSTARTER_TARGET_NAME  ${CPPSTARTER_LIB_NAME})

# Set required C/C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED TRUE)

option(CPPSTARTER_BUILD_SHARED "Build shared library" OFF)
option(CPPSTARTER_BUILD_EXAMPLES "Build the ${PROJECT_NAME} example applications" ${CPPSTARTER_IS_TOP_LEVEL})
option(CPPSTARTER_BUILD_TESTS "Build the ${PROJECT_NAME} test programs" ${CPPSTARTER_IS_TOP_LEVEL})
option(CPPSTARTER_INSTALL "Generate installation target" ${CPPSTARTER_IS_TOP_LEVEL})

set(CPPSTARTER_CMAKE_CONFIG_NAME "${PROJECT_NAME}Config")
set(CPPSTARTER_CMAKE_TARGET_NAME "${PROJECT_NAME}Target")

set(CPPSTARTER_INCLUDE_BUILD_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(CPPSTARTER_INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_INCLUDEDIR}")
set(CPPSTARTER_CONFIG_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/cmake/${PROJECT_NAME}" CACHE INTERNAL "Install directory path for config files.")

set(CPPSTARTER_CMAKE_CONFIG_TEMPLATE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
set(CPPSTARTER_CMAKE_CONFIG_TEMPLATE "${CPPSTARTER_CMAKE_CONFIG_TEMPLATE_DIR}/${CPPSTARTER_CMAKE_CONFIG_NAME}.cmake.in")
set(CPPSTARTER_CMAKE_VERSION_CONFIG_TEMPLATE "${CPPSTARTER_CMAKE_CONFIG_TEMPLATE_DIR}/${CPPSTARTER_CMAKE_CONFIG_NAME}Version.cmake.in")
set(CPPSTARTER_CMAKE_PACKAGE_CONFIG_TEMPLATE "${CPPSTARTER_CMAKE_CONFIG_TEMPLATE_DIR}/pkg-config.pc.in")

set(CPPSTARTER_CMAKE_CONFIG_DIR  ${CMAKE_CURRENT_BINARY_DIR})
set(CPPSTARTER_CMAKE_CONFIG_FILE  ${CPPSTARTER_CMAKE_CONFIG_DIR}/${PROJECT_NAME}Config.cmake)
set(CPPSTARTER_CMAKE_TARGETS_FILE  ${CPPSTARTER_CMAKE_CONFIG_DIR}/${PROJECT_NAME}Targets.cmake)
set(CPPSTARTER_CMAKE_PACKAGE_FILE  ${CPPSTARTER_CMAKE_CONFIG_DIR}/${PROJECT_NAME}.pc)
set(CPPSTARTER_CMAKE_VERSION_CONFIG_FILE  ${CPPSTARTER_CMAKE_CONFIG_DIR}/${PROJECT_NAME}ConfigVersion.cmake)

#--------------------------------------------------------------------
# Build library targets
#--------------------------------------------------------------------

if(CPPSTARTER_BUILD_SHARED)
    option(BUILD_SHARED_LIBS "Build shared libraries" ON)
endif()

add_library(${CPPSTARTER_LIB_NAME} ${CPPSTARTER_SOURCES})

target_include_directories(${CPPSTARTER_LIB_NAME} 
    PUBLIC 
        $<BUILD_INTERFACE:${CPPSTARTER_INCLUDE_BUILD_DIR}>
        $<INSTALL_INTERFACE:${CPPSTARTER_INCLUDE_INSTALL_DIR}>
)

#--------------------------------------------------------------------
# Configurations
#--------------------------------------------------------------------

# Default to build type "Release" unless tests are being built
if(NOT CMAKE_BUILD_TYPE)
	if (NOT CPPSTARTER_BUILD_TESTS)
		message(STATUS "No build type selected, default to Release")
		set(CMAKE_BUILD_TYPE "Release")
	else()
		message(STATUS "No build type selected but tests are being built, default to Debug")
		set(CMAKE_BUILD_TYPE "Debug")
	endif()
endif()

if(BUILD_SHARED_LIBS)
    set_target_properties(${CPPSTARTER_LIB_NAME} 
    PROPERTIES DEFINE_SYMBOL CPPSTARTER_BUILD_SHARED
    )
else()
    set_target_properties(${CPPSTARTER_LIB_NAME} 
    PROPERTIES DEFINE_SYMBOL CPPSTARTER_BUILD_STATIC
    )
endif()

# Output directories for a build tree
if(NOT DEFINED CMAKE_RUNTIME_OUTPUT_DIRECTORY)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
endif()

#--------------------------------------------------------------------
# Add Project subdirectories
#--------------------------------------------------------------------

# Build the example apps   
if(CPPSTARTER_BUILD_EXAMPLES)
    add_subdirectory(samples)
endif()

# Build the test apps   
if(CPPSTARTER_BUILD_TESTS)
    include(CTest)
    enable_testing()
    add_subdirectory(tests)

    # Set the unit test app project as the Visual Studio startup project
    # if the target compiler is some version of Microsoft Visual C++ and
    # if this project is the main project.
    if(MSVC AND CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
        set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT CPPSTARTERUnitTest)
    endif()
endif()

#--------------------------------------------------------------------
# Install a pkg-config file
#--------------------------------------------------------------------


#--------------------------------------------------------------------
# Configure package.
#--------------------------------------------------------------------
include(CMakePackageConfigHelpers)

configure_file(
    ${CPPSTARTER_CMAKE_CONFIG_TEMPLATE}
    ${CPPSTARTER_CMAKE_CONFIG_FILE}
    @ONLY
)

configure_file(
    ${CPPSTARTER_CMAKE_VERSION_CONFIG_TEMPLATE}
    ${CPPSTARTER_CMAKE_VERSION_CONFIG_FILE}
    @ONLY
)

configure_file(
    "${CPPSTARTER_CMAKE_PACKAGE_CONFIG_TEMPLATE}"
    "${CPPSTARTER_CMAKE_PACKAGE_FILE}"
)

#--------------------------------------------------------------------
# Install package.
#--------------------------------------------------------------------
if(CPPSTARTER_INSTALL)
    # Generate 'CPPSTARTERConfig.cmake', 'CPPSTARTERConfigVersion.cmake' and 'CPPSTARTERTargets.cmake'

    install(
        DIRECTORY ${CPPSTARTER_INCLUDE_BUILD_DIR}
        DESTINATION ${CPPSTARTER_INCLUDE_INSTALL_DIR}
    )

    install(
        FILES ${CPPSTARTER_CMAKE_CONFIG_FILE} ${CPPSTARTER_CMAKE_VERSION_CONFIG_FILE}
        DESTINATION ${CPPSTARTER_CONFIG_INSTALL_DIR}
    )

    export(
        TARGETS ${CPPSTARTER_TARGET_NAME}
        NAMESPACE ${PROJECT_NAME}::
        FILE ${CPPSTARTER_CMAKE_TARGETS_FILE}
    )

    install(
        TARGETS ${CPPSTARTER_TARGET_NAME}
        EXPORT ${CPPSTARTER_CMAKE_TARGET_NAME}
    )

    install(
        EXPORT ${CPPSTARTER_CMAKE_TARGET_NAME}
        NAMESPACE ${PROJECT_NAME}::
        DESTINATION ${CPPSTARTER_CONFIG_INSTALL_DIR}
    )

    install(
        FILES "${CPPSTARTER_CMAKE_PACKAGE_FILE}"
        DESTINATION ${CPPSTARTER_CONFIG_INSTALL_DIR}
    )

endif()
