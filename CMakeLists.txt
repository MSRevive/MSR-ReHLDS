cmake_minimum_required(VERSION 3.24)
project(msr_rehlds CXX)

add_custom_target(appversion DEPENDS
	COMMAND "${PROJECT_SOURCE_DIR}/src/version/appversion.sh" "${PROJECT_SOURCE_DIR}"
)

add_subdirectory(src)
add_subdirectory(src/dedicated)
add_subdirectory(src/filesystem)