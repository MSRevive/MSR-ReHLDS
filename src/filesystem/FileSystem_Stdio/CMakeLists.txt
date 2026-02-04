cmake_minimum_required(VERSION 3.24)
project(filesystem_stdio CXX)

option(DEBUG "Build with debug information." OFF)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		message(FATAL_ERROR "This project requires a 32-bit build. Please run cmake with '-A Win32'.")
	endif()

	# MSVC Flags
	# /W3 = Warning Level 3
	# /MP = Multi-processor compilation
	# /wd4996 = Disable deprecated warnings
	set(COMPILE_FLAGS "/W3 /MP /wd4996")

	if(DEBUG)
		set(COMPILE_FLAGS "${COMPILE_FLAGS} /MDd /Zi /Od")
	else()
		set(COMPILE_FLAGS "${COMPILE_FLAGS} /MD /O2 /Oi")
	endif()

else()
	set(CMAKE_SHARED_LIBRARY_CXX_FLAGS "")

	set(COMPILE_FLAGS "-m32 -U_FORTIFY_SOURCE -DFORTIFY_SOURCE=0")
	set(LINK_FLAGS "-m32")

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
	else()
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -mtune=generic -msse3 -fpermissive -fno-sized-deallocation -Wno-unknown-pragmas -Wno-unused-variable -Wno-unused-result -Wno-unused-function -Wno-write-strings -Wno-sign-compare -Wno-strict-aliasing")

		# Check if not Clang compiler AND GCC >= 8.3
		if (NOT "$ENV{CXX}" MATCHES "clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 8.0)
			set(COMPILE_FLAGS "${COMPILE_FLAGS} -Wno-stringop-truncation -Wno-class-memaccess")
		endif()
	endif()

	set(WRAP_FUNCS_LIST
		"scandir" "opendir" "freopen" "fopen" "fopen64" "open" "open64" "creat"
		"access" "stat" "lstat" "__xstat" "__lxstat" "__xstat64" "__lxstat64"
		"chmod" "chown" "lchown" "unlink" "symlink" "link" "mknod" "mount"
		"mkfifo" "rename" "utime" "utimes" "mkdir" "rmdir"
	)

	foreach(f ${WRAP_FUNCS_LIST})
		set(LINK_FLAGS "${LINK_FLAGS} -Wl,-wrap,${f}")
	endforeach()

	# GCC >= 8.3
	if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 8.0)
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -fcf-protection=none")
	endif()
endif()

set(PROJECT_SRC_DIR
	"${PROJECT_SOURCE_DIR}/src"
	"${PROJECT_SOURCE_DIR}/../.."
)

set(PROJECT_PUBLIC_DIR
	"${PROJECT_SOURCE_DIR}/../../common"
	"${PROJECT_SOURCE_DIR}/../../public"
	"${PROJECT_SOURCE_DIR}/../../public/rehlds"
)

set(FILESYSTEM_STDIO_SRCS
	"src/BaseFileSystem.cpp"
	"src/filesystem_helpers.cpp"
	"src/FileSystem_Stdio.cpp"
	"src/pathmatch.cpp"
	"src/public_amalgamation.cpp"
)

if(UNIX)
	list(APPEND FILESYSTEM_STDIO_SRCS "src/linux_support.cpp")
endif()

add_library(filesystem_stdio SHARED ${FILESYSTEM_STDIO_SRCS})

target_include_directories(filesystem_stdio PRIVATE
	${PROJECT_SRC_DIR}
	${PROJECT_PUBLIC_DIR}
)

if(MSVC)
	target_compile_definitions(filesystem_stdio PRIVATE
		WIN32
		_WINDOWS
		_WIN32
		_CRT_SECURE_NO_WARNINGS
	)
else()
	target_compile_definitions(filesystem_stdio PRIVATE
		_LINUX
		LINUX
		_GLIBCXX_USE_CXX11_ABI=0
		_strdup=strdup
		_stricmp=strcasecmp
		_strnicmp=strncasecmp
		_vsnprintf=vsnprintf
		_snprintf=snprintf
		_unlink=unlink
	)

	target_link_libraries(filesystem_stdio PRIVATE dl)
endif()

set_target_properties(filesystem_stdio PROPERTIES
	OUTPUT_NAME filesystem_stdio
	PREFIX ""
	COMPILE_FLAGS ${COMPILE_FLAGS}
	LINK_FLAGS ${LINK_FLAGS}
)

if(UNIX)
	set_target_properties(filesystem_stdio PROPERTIES POSITION_INDEPENDENT_CODE OFF)
endif()