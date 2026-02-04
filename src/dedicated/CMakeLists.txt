cmake_minimum_required(VERSION 3.24)
project(hlds CXX)

option(DEBUG "Build with debug information." OFF)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(PROJECT_SRC_DIR
	"${PROJECT_SOURCE_DIR}/src"
	"${PROJECT_SOURCE_DIR}/../"
)

set(PROJECT_PUBLIC_DIR
	"${PROJECT_SOURCE_DIR}/../engine"
	"${PROJECT_SOURCE_DIR}/../common"
	"${PROJECT_SOURCE_DIR}/../public"
	"${PROJECT_SOURCE_DIR}/../public/rehlds"
)

# Common sources for both platforms
set(DEDICATED_SRCS
	"src/dbg.cpp"
	"src/dedicated_exports.cpp"
	"src/public_amalgamation.cpp"
	"src/sys_ded.cpp"
	"src/vgui/vguihelpers.cpp"
)

set(COMMON_SRCS
	"../common/textconsole.cpp"
	"../common/SteamAppStartUp.cpp"
	"../common/ObjectList.cpp"
	"../common/commandline.cpp"
	"../common/minidump.cpp"
	"../engine/mem.cpp"
)

if(MSVC)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		message(WARNING "You are building for 64-bit, but HLDS is typically 32-bit. Please use a 32-bit generator (e.g., -A Win32).")
	endif()

	list(APPEND DEDICATED_SRCS "src/sys_win.cpp") 
	list(APPEND COMMON_SRCS "../common/TextConsoleWin32.cpp")

	# Compiler Flags
	# /W3 = Warning level 3
	# /MP = Multi-processor compilation
	# /wd4996 = Disable deprecated function warnings
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3 /MP /wd4996")

	# Disable exceptions to match Linux build
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHs-c-") 

	if(DEBUG)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Od /Zi")
	else()
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /O2")
	endif()

else()
	list(APPEND DEDICATED_SRCS "src/sys_linux.cpp")
	list(APPEND COMMON_SRCS "../common/TextConsoleUnix.cpp")

	# Avoid -rdynamic -fPIC options
	set(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "")

	set(COMPILE_FLAGS "-m32 -fno-pic -fno-pie -U_FORTIFY_SOURCE -DFORTIFY_SOURCE=0")
	set(LINK_FLAGS "-m32 -no-pie -Wl,--no-export-dynamic")

	set(COMPILE_FLAGS "${COMPILE_FLAGS} -Wall -fno-exceptions -fno-strict-aliasing -fno-strict-overflow")

	if (DEBUG)
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -g3 -O3 -ggdb")
	else()
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -g0 -O3 -fno-stack-protector")
	endif()

	# Check Intel C++ compiler
	if ("$ENV{CXX}" MATCHES "icpc")
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -Qoption,cpp,--treat_func_as_string_literal_cpp")
		set(LINK_FLAGS "${LINK_FLAGS} -static-intel -no-intel-extensions")

		if (NOT DEBUG)
			set(COMPILE_FLAGS "${COMPILE_FLAGS} -ipo")
			set(LINK_FLAGS "${COMPILE_FLAGS} -ipo")
		endif()
	else()
		set(COMPILE_FLAGS "${COMPILE_FLAGS} \
			-mtune=generic -msse3\
			-fpermissive -fno-sized-deallocation\
			-Wno-unused-result")

		# Check if not Clang compiler AND GCC >= 8.3
		if (NOT "$ENV{CXX}" MATCHES "clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 8.0)
			set(COMPILE_FLAGS "${COMPILE_FLAGS} -Wno-stringop-truncation -Wno-format-truncation")
		endif()
	endif()

	# GCC >= 8.3
	if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 8.0)
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -fcf-protection=none")
	endif()
endif()

if (NOT TARGET appversion AND UNIX)
	add_custom_target(appversion DEPENDS COMMAND "${PROJECT_SOURCE_DIR}/../version/appversion.sh" "${PROJECT_SOURCE_DIR}/../..")
endif()

add_executable(hlds ${DEDICATED_SRCS} ${COMMON_SRCS})

if(TARGET appversion)
	add_dependencies(hlds appversion)
endif()

target_include_directories(hlds PRIVATE
	${PROJECT_SRC_DIR}
	${PROJECT_PUBLIC_DIR}
)

if(MSVC)
	target_compile_definitions(hlds PRIVATE
		_CONSOLE
		WIN32
		_WIN32
		_CRT_SECURE_NO_WARNINGS
	)

else()
	target_compile_definitions(hlds PRIVATE
		LAUNCHER_FIXES
		_CONSOLE
		_LINUX
		LINUX
		_GLIBCXX_USE_CXX11_ABI=0
		_stricmp=strcasecmp
		_strnicmp=strncasecmp
		_strdup=strdup
		_vsnprintf=vsnprintf
	)

	target_link_libraries(hlds PRIVATE
		dl
	)
endif()

if(MSVC)
	set_target_properties(hlds PROPERTIES
		OUTPUT_NAME hlds
		PREFIX ""
		COMPILE_FLAGS ${COMPILE_FLAGS}
		LINK_FLAGS ${LINK_FLAGS}
	)
	
else()
	set_target_properties(hlds PROPERTIES
		OUTPUT_NAME hlds_linux
		PREFIX ""
		COMPILE_FLAGS ${COMPILE_FLAGS}
		LINK_FLAGS ${LINK_FLAGS}
		POSITION_INDEPENDENT_CODE OFF
	)
endif()