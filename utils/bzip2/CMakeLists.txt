cmake_minimum_required(VERSION 3.24)
project(bzip2 C)

set(BZIP2_SRCS
	"src/blocksort.c"
	"src/bzlib.c"
	"src/compress.c"
	"src/crctable.c"
	"src/decompress.c"
	"src/huffman.c"
	"src/precompiled.c"
	"src/randtable.c"
	"src/bzlib_private.h"
)

include_directories(
	"include"
)

if(MSVC)
	# Windows / MSVC Specific Flags
	# /O1  = Minimize size (Matches your original -O1)
	# /GS- = Buffer Security Check: No (Matches -fno-stack-protector)
	# /W3  = Warning Level 3
	# /wd4996 = Disable warnings about deprecated POSIX names (open, close, etc)
	add_compile_options(/O1 /GS- /W3 /wd4996)

	add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_WARNINGS)

else()
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g0 -O1 -fno-stack-protector -fno-strict-aliasing -fno-strict-overflow")
endif()

add_library(bzip2 STATIC ${BZIP2_SRCS})

if(UNIX)
	set_target_properties(bzip2 PROPERTIES
		COMPILE_FLAGS "-m32"
	)
endif()