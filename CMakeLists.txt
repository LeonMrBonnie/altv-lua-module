﻿# CMakeList.txt : Top-level CMake project file, do global configuration
# and include sub-projects here.
#
cmake_minimum_required (VERSION 3.8)

project ("altv-lua-module")

set(PROJECT_MODULE_NAME "lua-module")
if("${MODULE_TYPE}" STREQUAL "CLIENT")
	set(PROJECT_MODULE_NAME "lua-client-module")
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_definitions(-DMODULE_NAME="${PROJECT_MODULE_NAME}")
add_definitions(-DALT_${MODULE_TYPE}_API)
add_definitions(-D_CRT_SECURE_NO_WARNINGS)

if(UNIX)
	set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/linux64/${CMAKE_BUILD_TYPE})
else()
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/win64/${CMAKE_BUILD_TYPE})
	set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/win64/${CMAKE_BUILD_TYPE})
	# set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MD")
endif()
# set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MD")

file(GLOB_RECURSE PROJECT_SOURCE_FILES "./src/*.h" "./src/*.hpp" "./src/*.cpp")

include_directories(
	"${PROJECT_SOURCE_DIR}/src"
	"${PROJECT_SOURCE_DIR}/vendors/luajit/include"
	"${PROJECT_SOURCE_DIR}/vendors/alt-config"
	"${PROJECT_SOURCE_DIR}/vendors/cpp-sdk"
)

add_library(
	${PROJECT_MODULE_NAME} SHARED
	${PROJECT_SOURCE_FILES}
)

if("${MODULE_TYPE}" STREQUAL "CLIENT")
	add_library(
		${PROJECT_MODULE_NAME}-static STATIC
		${PROJECT_SOURCE_FILES}
	)
endif()

# target_compile_definitions(${PROJECT_MODULE_NAME} PRIVATE
# 	-DALT_SERVER_API
# )

if(UNIX)
	target_link_libraries(${PROJECT_MODULE_NAME} ${PROJECT_SOURCE_DIR}/vendors/luajit/lib/linux64/libluajit.so)
	#target_compile_options(${PROJECT_MODULE_NAME} PRIVATE
	#	-fuse-ld=lld
	#	-std=c++17
	#	-lstdc++fs
    #)
else()
	target_link_libraries(${PROJECT_MODULE_NAME}
	${PROJECT_SOURCE_DIR}/vendors/luajit/lib/win64/lua51.lib
	${PROJECT_SOURCE_DIR}/vendors/luajit/lib/win64/luajit.lib
	)
endif()