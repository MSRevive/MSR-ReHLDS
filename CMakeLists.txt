cmake_minimum_required(VERSION 3.24)
project(rehlds CXX)

if (WIN32)
	message(FATAL_ERROR "CMakeLists.txt Windows platform isn't supported yet. Use ReHLDS.sln instead it!")
endif()

add_subdirectory(src)
add_subdirectory(src/dedicated)
add_subdirectory(src/filesystem)