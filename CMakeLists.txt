cmake_minimum_required(VERSION 3.24)
project(rehlds CXX)

if (WIN32)
	message(FATAL_ERROR "CMakeLists.txt Windows platform isn't supported yet. Use ReHLDS.sln instead it!")
endif()

add_subdirectory(rehlds)
add_subdirectory(rehlds/dedicated)
add_subdirectory(rehlds/filesystem)