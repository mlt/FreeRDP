# FreeRDP: A Remote Desktop Protocol Implementation
# FreeRDP X11 Client
#
# Copyright 2012 Marc-Andre Moreau <marcandre.moreau@gmail.com>
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

set(MODULE_NAME "xfreerdp")
set(MODULE_PREFIX "FREERDP_CLIENT_X11")

include(FindXmlto)
include_directories(${X11_INCLUDE_DIRS})

set(${MODULE_PREFIX}_SRCS
	xf_gdi.c
	xf_gdi.h
	xf_rail.c
	xf_rail.h
	xf_tsmf.c
	xf_tsmf.h
	xf_event.c
	xf_event.h
	xf_cliprdr.c
	xf_cliprdr.h
	xf_monitor.c
	xf_monitor.h
	xf_graphics.c
	xf_graphics.h
	xf_keyboard.c
	xf_keyboard.h
	xf_window.c
	xf_window.h
	xfreerdp.c
	xfreerdp.h)

add_executable(${MODULE_NAME} ${${MODULE_PREFIX}_SRCS})

set(${MODULE_PREFIX}_LIBS
	${X11_LIBRARIES}
	${CMAKE_DL_LIBS})

if(WITH_MANPAGES)
	if(XMLTO_FOUND)
		add_custom_command(OUTPUT xfreerdp.1
			COMMAND ${XMLTO_EXECUTABLE} man ${CMAKE_CURRENT_SOURCE_DIR}/xfreerdp.1.xml
			DEPENDS xfreerdp.1.xml)

		add_custom_target(xfreerdp.manpage ALL
			DEPENDS xfreerdp.1)

		install(FILES ${CMAKE_CURRENT_BINARY_DIR}/xfreerdp.1 DESTINATION share/man/man1)
	else(XMLTO_FOUND)
		message(WARNING "WITH_MANPAGES was set, but xmlto was not found. man-pages will not be installed")
	endif(XMLTO_FOUND)
endif(WITH_MANPAGES)

set(XSHM_FEATURE_TYPE "REQUIRED")
set(XSHM_FEATURE_PURPOSE "X11 shared memory")
set(XSHM_FEATURE_DESCRIPTION "X11 shared memory extension")

set(XINERAMA_FEATURE_TYPE "RECOMMENDED")
set(XINERAMA_FEATURE_PURPOSE "multi-monitor")
set(XINERAMA_FEATURE_DESCRIPTION "X11 multi-monitor extension")

set(XEXT_FEATURE_TYPE "RECOMMENDED")
set(XEXT_FEATURE_PURPOSE "X11 extension")
set(XEXT_FEATURE_DESCRIPTION "X11 core extensions")

set(XCURSOR_FEATURE_TYPE "RECOMMENDED")
set(XCURSOR_FEATURE_PURPOSE "cursor")
set(XCURSOR_FEATURE_DESCRIPTION "X11 cursor extension")

set(XV_FEATURE_TYPE "RECOMMENDED")
set(XV_FEATURE_PURPOSE "video")
set(XV_FEATURE_DESCRIPTION "X11 video extension")

find_feature(XShm ${XSHM_FEATURE_TYPE} ${XSHM_FEATURE_PURPOSE} ${XSHM_FEATURE_DESCRIPTION})
find_feature(Xinerama ${XINERAMA_FEATURE_TYPE} ${XINERAMA_FEATURE_PURPOSE} ${XINERAMA_FEATURE_DESCRIPTION})
find_feature(Xext ${XEXT_FEATURE_TYPE} ${XEXT_FEATURE_PURPOSE} ${XEXT_FEATURE_DESCRIPTION})
find_feature(Xcursor ${XCURSOR_FEATURE_TYPE} ${XCURSOR_FEATURE_PURPOSE} ${XCURSOR_FEATURE_DESCRIPTION})
find_feature(Xv ${XV_FEATURE_TYPE} ${XV_FEATURE_PURPOSE} ${XV_FEATURE_DESCRIPTION})

if(WITH_XINERAMA)
	add_definitions(-DWITH_XINERAMA)
	include_directories(${XINERAMA_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XINERAMA_LIBRARIES})
endif()

if(WITH_XEXT)
	add_definitions(-DWITH_XEXT)
	include_directories(${XEXT_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XEXT_LIBRARIES})
endif()

if(WITH_XCURSOR)
	add_definitions(-DWITH_XCURSOR)
	include_directories(${XCURSOR_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XCURSOR_LIBRARIES})
endif()

if(WITH_XV)
	add_definitions(-DWITH_XV)
	include_directories(${XV_INCLUDE_DIRS})
	set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${XV_LIBRARIES})
endif()

include_directories(${CMAKE_SOURCE_DIR}/resources)

set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} freerdp-client)

set_complex_link_libraries(VARIABLE ${MODULE_PREFIX}_LIBS MONOLITHIC ${MONOLITHIC_BUILD}
	MODULE freerdp
	MODULES freerdp-core freerdp-gdi freerdp-locale freerdp-rail freerdp-utils)

target_link_libraries(${MODULE_NAME} ${${MODULE_PREFIX}_LIBS})

install(TARGETS ${MODULE_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})

set_property(TARGET ${MODULE_NAME} PROPERTY FOLDER "Client/X11")
