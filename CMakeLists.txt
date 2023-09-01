cmake_minimum_required(VERSION 3.0.0...3.5)

project(lua C)

set(lua_MAJOR_VERSION 5)
set(lua_MINOR_VERSION 4)
set(lua_MICRO_VERSION 6)
set(lua_VERSION "${lua_MAJOR_VERSION}.${lua_MINOR_VERSION}.${lua_MICRO_VERSION}")
set(VERSION "${lua_VERSION}")

message(STATUS "[lua] VERSION: ${lua_VERSION}")

option(lua_SHARED "Build ${PROJ_NAME} as a shared lib" ON)
option(lua_STATIC "Build ${PROJ_NAME} as a shared lib" ON)
option(lua_SKIP_INSTALL "" OFF)
option(lua_ONELUA "" OFF)
set(lua_CMAKE_DEBUG_POSTFIX "d" CACHE STRING "Name suffix for debug builds")

message(STATUS "[lua] lua_SHARED: ${lua_SHARED}")
message(STATUS "[lua] lua_STATIC: ${lua_STATIC}")
message(STATUS "[lua] lua_SKIP_INSTALL: ${lua_SKIP_INSTALL}")
message(STATUS "[lua] lua_ONELUA: ${lua_ONELUA}")

set(lua_PUBLIC_H_FILES
	lauxlib.h
	lua.h
	luaconf.h
	lualib.h
)
set(lua_PRIVATE_H_FILES
	lapi.h lcode.h lctype.h ldebug.h ldo.h
	lfunc.h lgc.h ljumptab.h llex.h llimits.h
	lmem.h lobject.h lopcodes.h lopnames.h lparser.h
	lprefix.h lstate.h lstring.h ltable.h ltests.h
	ltm.h lundump.h lvm.h lzio.h
)
set(lua_SRC_FILES
	lapi.c lauxlib.c lbaselib.c lcode.c lcorolib.c
	lctype.c ldblib.c ldebug.c ldo.c ldump.c
	lfunc.c lgc.c linit.c liolib.c llex.c
	lmathlib.c lmem.c loadlib.c lobject.c lopcodes.c
	loslib.c lparser.c lstate.c lstring.c lstrlib.c
	ltable.c ltablib.c ltests.c ltm.c lua.c
	lundump.c lutf8lib.c lvm.c lzio.c
)

set(lua_ONELUA_C_FILES onelua.c)

if (WIN32 AND NOT MINGW)
	set(lua_INSTALL_CMAKE_DIR_ROOT_DEFAULT "cmake")
else()
	set(lua_INSTALL_CMAKE_DIR_ROOT_DEFAULT "${CMAKE_INSTALL_LIBDIR}/cmake")
endif()
set(lua_INSTALL_CMAKE_DIR_ROOT "${lua_INSTALL_CMAKE_DIR_ROOT_DEFAULT}" CACHE STRING "Root folder where to install cmake files")

if (WIN32 AND NOT MINGW)
	set(lua_INSTALL_CMAKE_DIR "${lua_INSTALL_CMAKE_DIR_ROOT}")
else()
	set(lua_INSTALL_CMAKE_DIR "${lua_INSTALL_CMAKE_DIR_ROOT}/lua")
endif()

if (WIN32 AND NOT MINGW)
	set(lua_INSTALL_INCLUDEDIR_DEFAULT "include/lua")
else()
	set(lua_INSTALL_INCLUDEDIR_DEFAULT "${CMAKE_INSTALL_INCLUDEDIR}/include/lua")
endif()
set(lua_INSTALL_INCLUDEDIR "${lua_INSTALL_INCLUDEDIR_DEFAULT}" CACHE STRING "Folder to install public header files")

if (lua_SHARED)
	add_library(lua SHARED)
	target_sources(lua PRIVATE ${lua_PUBLIC_H_FILES})
	if (NOT lua_ONELUA)
		target_sources(lua PRIVATE ${lua_PRIVATE_H_FILES})
		target_sources(lua PRIVATE ${lua_SRC_FILES})
	else()
		target_sources(lua PRIVATE ${lua_ONELUA_C_FILES})
	endif()
	set_target_properties(lua PROPERTIES
		ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
		LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
		RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
		FOLDER "lua"
	)
	target_include_directories(lua
		PRIVATE
			"$<BUILD_INTERFACE:${lua_SOURCE_DIR}>"
			"$<BUILD_INTERFACE:${lua_BINARY_DIR}>"
		PUBLIC
			"$<BUILD_INTERFACE:${lua_SOURCE_DIR}>"
			"$<BUILD_INTERFACE:${lua_BINARY_DIR}>"
			"$<INSTALL_INTERFACE:${lua_INSTALL_INCLUDEDIR}>"
	)
	add_library(lua::lua ALIAS lua)
endif()

if (lua_STATIC)
	add_library(lua-static STATIC)
	target_sources(lua-static PRIVATE ${lua_PUBLIC_H_FILES})
	if (NOT lua_ONELUA)
		target_sources(lua-static PRIVATE ${lua_PRIVATE_H_FILES})
		target_sources(lua-static PRIVATE ${lua_SRC_FILES})
	else()
		target_sources(lua-static PRIVATE ${lua_ONELUA_C_FILES})
	endif()
	set_target_properties(lua-static PROPERTIES
		ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
		LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
		RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
		FOLDER "lua-static"
	)
	target_include_directories(lua-static
		PRIVATE
			"$<BUILD_INTERFACE:${lua_SOURCE_DIR}>"
			"$<BUILD_INTERFACE:${lua_BINARY_DIR}>"
		PUBLIC
			"$<BUILD_INTERFACE:${lua_SOURCE_DIR}>"
			"$<BUILD_INTERFACE:${lua_BINARY_DIR}>"
			"$<INSTALL_INTERFACE:${lua_INSTALL_INCLUDEDIR}>"
	)
	add_library(lua::lua-static ALIAS lua-static)
endif()

if (NOT lua_SKIP_INSTALL)
	if (lua_SHARED)
		install(TARGETS lua EXPORT luaSharedTargets
			PUBLIC_HEADER DESTINATION "${lua_INSTALL_INCLUDEDIR}"
			ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
			LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
			RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
			INCLUDES DESTINATION "${lua_INSTALL_INCLUDEDIR}"
			FRAMEWORK DESTINATION "."
		)
	endif()
	
	if (lua_STATIC)
		install(TARGETS lua-static EXPORT luaStaticTargets
			PUBLIC_HEADER DESTINATION "${lua_INSTALL_INCLUDEDIR}"
			ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
			LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
			RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
			INCLUDES DESTINATION "${lua_INSTALL_INCLUDEDIR}"
			FRAMEWORK DESTINATION "."
		)
	endif()
	
	if (lua_SHARED)
		install(EXPORT luaSharedTargets
			FILE "luaSharedTargets.cmake"
			NAMESPACE lua::
			DESTINATION "${lua_INSTALL_CMAKE_DIR}"
		)
	endif()
	
	if (lua_STATIC)
		install(EXPORT luaStaticTargets
			FILE "luaStaticTargets.cmake"
			NAMESPACE lua::
			DESTINATION "${lua_INSTALL_CMAKE_DIR}"
		)
	endif()
	
	install(FILES ${lua_PUBLIC_H_FILES} DESTINATION "${lua_INSTALL_INCLUDEDIR}")
endif()