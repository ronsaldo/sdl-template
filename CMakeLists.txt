# Leng cmake build script.
cmake_minimum_required(VERSION 2.8)
project(SdlTemplate)

# Check the build type
if (CMAKE_BUILD_TYPE STREQUAL "")
	# CMake defaults to leaving CMAKE_BUILD_TYPE empty. This screws up
	# differentiation between debug and release builds.
	set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build, options are: None (CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif ()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Output paths.
set(CMAKE_DEBUG_POSTFIX "_d")
 
set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/dist")
set(SdlTemplate_DATA_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/dist/data")
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/;${CMAKE_MODULE_PATH}")

# Turn warnings and use C++ 2011.
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
	# using Visual Studio C++
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
else()
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++11 -pthread")
	
	# Export symbols from applications.
	#set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--export-dynamic")
endif()

# Perform platform checks
include(${CMAKE_ROOT}/Modules/CheckIncludeFile.cmake)
include(${CMAKE_ROOT}/Modules/CheckIncludeFileCXX.cmake)
include(${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
include(${CMAKE_ROOT}/Modules/CheckLibraryExists.cmake)

# Set output dir.
set(EXECUTABLE_OUTPUT_PATH "${SdlTemplate_BINARY_DIR}/dist")
set(LIBRARY_OUTPUT_PATH "${SdlTemplate_BINARY_DIR}/dist")

# Use pkg-config.
find_package(PkgConfig)
if(PKG_CONFIG_FOUND)

    # Find SDL2
    pkg_check_modules(SDL2 REQUIRED sdl2)
    include_directories(${SDL2_INCLUDE_DIRS})

    # Delaunay dependencies libraries
    set(SdlTemplate_DEP_LIBS ${SDL2_LIBRARIES} ${TerrainLod_DEP_LIBS})

else()
    # For VisualStudio or mingw in Window
    find_path(SDL2_INCLUDE_DIR
      NAMES SDL.h
    )

    find_path(SDL2_LIBRARY_PATH
      NAMES SDL2.lib SDL2.a
    )

    find_library(SDL2_LIBRARY NAMES SDL2 PATHS ${SDL2_LIBRARY_PATH})
    if(WIN32)
	find_library(SDL2_MAIN_LIBRARY NAMES SDL2main PATHS ${SDL2_LIBRARY_PATH})
    else()
	set(SDL2_MAIN_LIBRARY "")
    endif()
    
    set(TerrainLod_DEP_LIBS ${TerrainLod_DEP_LIBS} ${SDL2_LIBRARY} ${SDL2_MAIN_LIBRARY})
    include_directories("${SDL2_INCLUDE_DIR}")
endif()

# Add the current directory.
include_directories(
.
"${PROJECT_SOURCE_DIR}/include"
"${PROJECT_SOURCE_DIR}/thirdparty/glm"
)

# Build the program
add_subdirectory(src)
