if(NOT CMAKE_GENERATOR MATCHES "Ninja" AND UNIX)
  message(FATAL_ERROR "Need Ninja on Unix")
endif()

# cotire example project

add_library(foo ${CMAKE_CURRENT_BINARY_DIR}/foo.c ${CMAKE_CURRENT_BINARY_DIR}/foo.h)
include_directories(foo PUBLIC ${CMAKE_CURRENT_BINARY_DIR})

add_custom_command(
	OUTPUT foo.c foo.h
	COMMAND sleep 5
	COMMAND touch foo.c foo.h)

add_executable(example main.cpp example.cpp log.cpp log.h example.h)
add_dependencies(example foo)

# enable warnings
if (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
	set_target_properties(example PROPERTIES COMPILE_FLAGS "/Wall")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
	set_target_properties(example PROPERTIES COMPILE_FLAGS "-Weverything")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
	set_target_properties(example PROPERTIES COMPILE_FLAGS "-Wall -Wextra")
endif()

set_target_properties(example PROPERTIES COTIRE_CXX_PREFIX_HEADER_INIT "stdafx.h")
set_target_properties(example PROPERTIES COTIRE_ADD_UNITY_BUILD FALSE)
cotire(example)

# cotire sets the following properties
get_target_property(_unitySource example COTIRE_CXX_UNITY_SOURCE)
get_target_property(_prefixHeader example COTIRE_CXX_PREFIX_HEADER)
get_target_property(_precompiledHeader example COTIRE_CXX_PRECOMPILED_HEADER)
get_target_property(_unityTargetName example COTIRE_UNITY_TARGET_NAME)

if (_unitySource)
	message(STATUS "example unity source: ${_unitySource}")
endif()
if (_prefixHeader)
	message(STATUS "example prefix header: ${_prefixHeader}")
endif()
if (_precompiledHeader)
	message(STATUS "example precompiled header: ${_precompiledHeader}")
endif()
if (TARGET ${_unityTargetName})
	message(STATUS "example unity target: ${_unityTargetName}")
endif()
