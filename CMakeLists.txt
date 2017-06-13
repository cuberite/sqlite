
cmake_minimum_required (VERSION 2.6)
project (sqlite)

include_directories ("${PROJECT_SOURCE_DIR}/../")

file(GLOB SOURCE
    "*.c"
)


# Lua is required as a DLL for LuaSQLite:
if (WIN32)
	add_definitions(-DLUA_BUILD_AS_DLL)
endif()


# add headers to MSVC project files:
if (MSVC)
	file(GLOB HEADERS "src/*.h")
	list(REMOVE_ITEM SOURCE "${PROJECT_SOURCE_DIR}/src/lua.h" "${PROJECT_SOURCE_DIR}/src/luac.h")
	set(SOURCE ${SOURCE} ${HEADERS})
	source_group("Sources" FILES ${SOURCE})
endif()

# FreeBSD requires us to define this to get POSIX 2001 standard
if (${CMAKE_SYSTEM_NAME} STREQUAL "FreeBSD")
	add_flags_cxx("-D_XOPEN_SOURCE=600")
endif()


macro(set_exe_flags)
	if (NOT MSVC)
		if ("${CLANG_VERSION}" VERSION_GREATER 3.5)
			include(CheckCXXCompilerFlag)
			check_cxx_compiler_flag(-Wno-reserved-id-macro HAS_NO_RESERVED_ID_MACRO)
			check_cxx_compiler_flag(-Wno-documentation-unknown-command HAS_NO_DOCUMENTATION_UNKNOWN)
			if (HAS_NO_RESERVED_ID_MACRO)
				# Use this flag to ignore error for a reserved macro problem in sqlite 3
				add_flags_cxx("-Wno-reserved-id-macro")
			endif()
			if (HAS_NO_DOCUMENTATION_UNKNOWN)
				# Ignore another problem in sqlite
				add_flags_cxx("-Wno-documentation-unknown-command")
			endif()
		endif()
	endif()
endmacro()


add_library(sqlite ${SOURCE})
target_link_libraries(sqlite lua)

if (UNIX)
	target_link_libraries(sqlite ${DYNAMIC_LOADER})
endif()
