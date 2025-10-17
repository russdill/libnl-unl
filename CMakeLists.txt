CMAKE_MINIMUM_REQUIRED(VERSION 3.10)

PROJECT(libnl-unl C)
INCLUDE(GNUInstallDirs)

INCLUDE_DIRECTORIES(include)

find_package(PkgConfig REQUIRED)
pkg_check_modules(nl3 REQUIRED libnl-3.0)
pkg_check_modules(nl3-genl REQUIRED libnl-genl-3.0)

include_directories(${nl3_INCLUDE_DIRS} ${nl3-genl_INCLUDE_DIRS})
link_directories(${nl3_LIBRARY_DIRS} ${nl3-genl_LIBRARY_DIRS})

ADD_DEFINITIONS(-Wall -Werror -Wextra -Wno-unused-parameter)
STRING(REPLACE "-DNDEBUG" "" CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}")

SET(SOURCES unl.c)

SET(SOVERSION 1 CACHE STRING "Override libnl-unl library version")

ADD_LIBRARY(nl-unl SHARED ${SOURCES})
SET_TARGET_PROPERTIES(nl-unl PROPERTIES SOVERSION ${SOVERSION})

target_link_libraries(nl-unl ${nl3_LIBRARIES} ${nl3-genl_LIBRARIES})

CONFIGURE_FILE(
	"${CMAKE_CURRENT_SOURCE_DIR}/libnl-unl.pc.in"
	"${CMAKE_CURRENT_BINARY_DIR}/libnl-unl.pc"
	@ONLY
)

INSTALL(TARGETS nl-unl LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
INSTALL(FILES ${CMAKE_BINARY_DIR}/libnl-unl.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
INSTALL(
	DIRECTORY ${CMAKE_SOURCE_DIR}/include/
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libnl-unl
	FILES_MATCHING PATTERN "*.h"
)

ADD_CUSTOM_TARGET(debian
        COMMAND ${CMAKE_COMMAND} -E echo "Generating debian/changelog from git..."
        COMMAND ${CMAKE_SOURCE_DIR}/debian/generate-changelog.sh
        COMMAND dpkg-buildpackage -b -uc -us
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Building Debian package"
)
