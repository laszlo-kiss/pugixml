# vim: et ts=4 sts=4 sw=4 tw=0

CMAKE_MINIMUM_REQUIRED(VERSION 2.8.5)
PROJECT(pugixml)
ENABLE_TESTING()

OPTION(PUGIXML_WITH_CMAKE_PACKAGE "Generate and install cmake package files" OFF)

# Ensures that CMAKE_BUILD_TYPE is visible in cmake-gui on Unix
IF(NOT WIN32)
    IF(NOT CMAKE_BUILD_TYPE)
        SET(CMAKE_BUILD_TYPE Release CACHE STRING
            "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel Coverage."
            FORCE)
    ENDIF(NOT CMAKE_BUILD_TYPE)
ENDIF(NOT WIN32)

SET(LIB_SUFFIX "" CACHE STRING "Optional arch-dependent suffix for the library installation directory")

SET(RUNTIME_INSTALL_DIR bin
    CACHE PATH "Install dir for executables and dlls")
SET(ARCHIVE_INSTALL_DIR lib${LIB_SUFFIX}
    CACHE PATH "Install dir for static libraries")
SET(LIBRARY_INSTALL_DIR lib${LIB_SUFFIX}
    CACHE PATH "Install dir for shared libraries")
SET(INCLUDE_INSTALL_DIR include
    CACHE PATH "Install dir for headers")
SET(PACKAGE_INSTALL_DIR lib${LIB_SUFFIX}/cmake
    CACHE PATH "Install dir for cmake package config files")
MARK_AS_ADVANCED( RUNTIME_INSTALL_DIR ARCHIVE_INSTALL_DIR INCLUDE_INSTALL_DIR PACKAGE_INSTALL_DIR )

# Set variable named ${VAR_NAME} to value ${VALUE}
FUNCTION(set_using_dynamic_name VAR_NAME VALUE)
    SET( "${VAR_NAME}" "${VALUE}" PARENT_SCOPE)
ENDFUNCTION(set_using_dynamic_name)

# Extract major, minor, patch from version text
# Parse a version string "X.Y.Z" and outputs
# version parts in ${OUPUT_PREFIX}_MAJOR, _MINOR, _PATCH.
# If parse succeeds then ${OUPUT_PREFIX}_FOUND is TRUE.
MACRO(pugixml_parse_version VERSION_TEXT OUPUT_PREFIX)
    SET(VERSION_REGEX "[0-9]+\\.[0-9]+\\.[0-9]+(-[a-zA-Z0-9_]+)?")
    IF( ${VERSION_TEXT} MATCHES ${VERSION_REGEX} )
        STRING(REGEX MATCHALL "[0-9]+|-([A-Za-z0-9_]+)" VERSION_PARTS ${VERSION_TEXT})
        LIST(GET VERSION_PARTS 0 ${OUPUT_PREFIX}_MAJOR)
        LIST(GET VERSION_PARTS 1 ${OUPUT_PREFIX}_MINOR)
        LIST(GET VERSION_PARTS 2 ${OUPUT_PREFIX}_PATCH)
        set_using_dynamic_name( "${OUPUT_PREFIX}_FOUND" TRUE )
    ELSE( ${VERSION_TEXT} MATCHES ${VERSION_REGEX} )
        set_using_dynamic_name( "${OUPUT_PREFIX}_FOUND" FALSE )
    ENDIF( ${VERSION_TEXT} MATCHES ${VERSION_REGEX} )
ENDMACRO(pugixml_parse_version)

# Read out version from "version" file
FILE(STRINGS "version" PUGIXML_VERSION)
SET( PUGIXML_VERSION_MAJOR X )
SET( PUGIXML_VERSION_MINOR Y )
SET( PUGIXML_VERSION_PATCH Z )
#SET( PUGIXML_VERSION 1.5.0 )
pugixml_parse_version( ${PUGIXML_VERSION} PUGIXML_VERSION )
IF(NOT PUGIXML_VERSION_FOUND)
    MESSAGE(FATAL_ERROR "Failed to parse version string properly. Expect X.Y.Z")
ENDIF(NOT PUGIXML_VERSION_FOUND)

MESSAGE(STATUS "PugiXML Version: ${PUGIXML_VERSION_MAJOR}.${PUGIXML_VERSION_MINOR}.${PUGIXML_VERSION_PATCH}")

macro(UseCompilationWarningAsError)
	if ( MSVC )
        # Only enabled in debug because some old versions of VS STL generate
        # warnings when compiled in release configuration.
		set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /WX ")
	endif( MSVC )
endmacro()

# Include our configuration header
#INCLUDE_DIRECTORIES( ${pugixml_SOURCE_DIR}/include )

if ( MSVC )
    # Only enabled in debug because some old versions of VS STL generate
    # unreachable code warning when compiled in release configuration.
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /W4 ")
endif( MSVC )

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  # using regular Clang or AppleClang
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  # using GCC
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -Wall -Wextra -pedantic")
endif()

IF(PUGIXML_WITH_WARNING_AS_ERROR)
    UseCompilationWarningAsError()
ENDIF(PUGIXML_WITH_WARNING_AS_ERROR)

#IF(PUGIXML_WITH_PKGCONFIG_SUPPORT)
#	CONFIGURE_FILE(
#		"pkg-config/pugixml.pc.in"
#		"pkg-config/pugixml.pc"
#		@ONLY)
#	INSTALL(FILES "${CMAKE_BINARY_DIR}/pkg-config/pugixml.pc"
#		DESTINATION "${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}/pkgconfig")
#ENDIF(PUGIXML_WITH_PKGCONFIG_SUPPORT)

IF(PUGIXML_WITH_CMAKE_PACKAGE)
        INSTALL(EXPORT pugixml
                DESTINATION ${PACKAGE_INSTALL_DIR}/pugixml
                FILE        pugixmlConfig.cmake)
ENDIF(PUGIXML_WITH_CMAKE_PACKAGE)

# Build the different applications
ADD_SUBDIRECTORY( src )

#install the includes
#ADD_SUBDIRECTORY( include )
