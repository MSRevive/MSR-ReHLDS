#----------------------------------------
# 1. Preparing build:
#    rm -rf build
#    mkdir build && cd build
#
# 2. Select compiler and build it
# - Compile with Clang:
#    CC="clang" CXX="clang++" cmake ..
#    make
#
# - Compile with Intel C++ Compiler:
#    CC="icc" CXX="icpc" cmake ..
#    make
#
# - Compile with GCC Compiler:
#    cmake ..
#    make
#
# - Windows (MSVC):
#    cmake -A Win32 ..
#----------------------------------------

cmake_minimum_required(VERSION 3.24)
project(engine CXX)

option(DEBUG "Build with debug information." OFF)
option(USE_STATIC_LIBSTDC "Enables static linking libstdc++." OFF)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		message(FATAL_ERROR "This project requires a 32-bit build. Please run cmake with '-A Win32'.")
	endif()

	# MSVC Flags:
	# /W3 = Warning level 3
	# /MP = Multi-processor compilation
	# /GR- = Disable RTTI (matches -fno-rtti)
	# /wd4996 = Disable deprecated warnings (unsafe CRT functions)
	set(COMPILE_FLAGS "/W3 /MP /GR- /wd4996")
	
	add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_WARNINGS)

	if(DEBUG)
		set(COMPILE_FLAGS "${COMPILE_FLAGS} /MDd /Zi /Od")
	else()
		set(COMPILE_FLAGS "${COMPILE_FLAGS} /MD /O2 /Oi")
	endif()

	if(USE_STATIC_LIBSTDC)
		string(REPLACE "/MD" "/MT" COMPILE_FLAGS ${COMPILE_FLAGS})
	endif()

else()
	set(CMAKE_SHARED_LIBRARY_CXX_FLAGS "")
	set(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "")

	set(COMPILE_FLAGS "-m32 -U_FORTIFY_SOURCE -DFORTIFY_SOURCE=0")
	set(LINK_FLAGS "-m32")

	set(COMPILE_FLAGS "${COMPILE_FLAGS} -Wall -fno-rtti -fno-exceptions -fno-strict-aliasing -fno-strict-overflow")

	set(COMPILE_FLAGS "${COMPILE_FLAGS} -ffunction-sections -fdata-sections")

	if (DEBUG)
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -g3 -O3 -ggdb")
	else()
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -g0 -O3 -fno-stack-protector")
	endif()

	# Check Intel C++ compiler
	if ("$ENV{CXX}" MATCHES "icpc")
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -fp-model=precise -fasm-blocks -Qoption,cpp,--treat_func_as_string_literal_cpp")
		set(LINK_FLAGS "${LINK_FLAGS} -static-intel -no-intel-extensions")

		if (NOT DEBUG)
			set(COMPILE_FLAGS "${COMPILE_FLAGS} -ipo")
			set(LINK_FLAGS "${LINK_FLAGS} -ipo")
		endif()
	else()
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -mtune=generic -msse3 -fpermissive -fno-sized-deallocation -Wno-unknown-pragmas -Wno-invalid-offsetof -Wno-unused-variable -Wno-unused-result -Wno-unused-function -Wno-delete-non-virtual-dtor -Wno-write-strings -Wno-format -Wno-sign-compare -Wno-strict-aliasing -Wno-ignored-attributes")

		# Check if not Clang compiler
		if (NOT "$ENV{CXX}" MATCHES "clang")
			set(COMPILE_FLAGS "${COMPILE_FLAGS} -fno-plt -Wno-unused-but-set-variable")
			# GCC >= 8.3
			if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 8.0)
				set(COMPILE_FLAGS "${COMPILE_FLAGS} -Wno-stringop-truncation -Wno-format-truncation -Wno-class-memaccess")
			endif()
		endif()
	endif()

	# GCC >= 8.3
	if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 8.0)
		set(COMPILE_FLAGS "${COMPILE_FLAGS} -fcf-protection=none")
	endif()

	if (NOT DEBUG)
		set(LINK_FLAGS "${LINK_FLAGS} -Wl,-gc-sections -Wl,--version-script=\"${PROJECT_SOURCE_DIR}/../version_script.lds\"")
	endif()
endif()

set(PROJECT_SRC_DIR
	"${PROJECT_SOURCE_DIR}"
	"${PROJECT_SOURCE_DIR}/engine"
	"${PROJECT_SOURCE_DIR}/common"
	"${PROJECT_SOURCE_DIR}/pm_shared"
	"${PROJECT_SOURCE_DIR}/rehlds"
)

set(PROJECT_BZIP2_DIR
	"${PROJECT_SOURCE_DIR}/../utils/bzip2/include"
)

set(PROJECT_PUBLIC_DIR
	"${PROJECT_SOURCE_DIR}/public"
	"${PROJECT_SOURCE_DIR}/public/rehlds"
)

set(ENGINE_SRCS
	engine/sv_main.cpp
	engine/sv_user.cpp
	engine/sv_phys.cpp
	engine/sv_move.cpp
	engine/sv_pmove.cpp
	engine/sv_log.cpp
	engine/sv_remoteaccess.cpp
	engine/sv_steam3.cpp
	engine/sv_upld.cpp
	engine/sys_dll.cpp
	engine/sys_dll2.cpp
	engine/sys_engine.cpp
	engine/sys_linuxwind.cpp
	engine/SystemWrapper.cpp
	engine/host.cpp
	engine/host_cmd.cpp
	engine/net_chan.cpp
	engine/net_ws.cpp
	engine/pmove.cpp
	engine/pmovetst.cpp
	engine/pr_cmds.cpp
	engine/pr_edict.cpp
	engine/wad.cpp
	engine/model.cpp
	engine/world.cpp
	engine/zone.cpp
	engine/cmd.cpp
	engine/cmodel.cpp
	engine/com_custom.cpp
	engine/common.cpp
	engine/crc.cpp
	engine/cvar.cpp
	engine/decals.cpp
	engine/delta.cpp
	engine/delta_jit.cpp
	engine/ed_strpool.cpp
	engine/filesystem.cpp
	engine/filesystem_internal.cpp
	engine/hashpak.cpp
	engine/info.cpp
	engine/ipratelimit.cpp
	engine/l_studio.cpp
	engine/textures.cpp
	engine/tmessage.cpp
	engine/traceinit.cpp
	engine/unicode_strtools.cpp
	engine/buildnum.cpp
	engine/mathlib.cpp
	engine/mathlib_sse.cpp
	engine/md5.cpp
	engine/mem.cpp
	engine/module.cpp
	engine/r_studio.cpp
	engine/vid_null.cpp
	engine/cl_null.cpp
	engine/snd_null.cpp
	engine/sse_mathfun.cpp
	engine/public_amalgamation.cpp
	rehlds/flight_recorder.cpp
	rehlds/FlightRecorderImpl.cpp
	rehlds/hookchains_impl.cpp
	rehlds/main.cpp
	rehlds/platform.cpp
	rehlds/public_amalgamation.cpp
	rehlds/rehlds_api_impl.cpp
	rehlds/rehlds_interfaces_impl.cpp
	rehlds/rehlds_messagemngr_impl.cpp
	rehlds/rehlds_security.cpp
)

set(COMMON_SRCS
	"common/BaseSystemModule.cpp"
	"common/ObjectList.cpp"
	"common/TokenLine.cpp"
)

set(PUBLIC_SRCS
	"public/tier0/dbg.cpp"
	"public/registry.cpp"
	"public/steamid.cpp"
	"public/utlbuffer.cpp"
)

add_library(engine SHARED ${ENGINE_SRCS})

if (NOT TARGET bzip2)
	add_subdirectory(../dep/bzip2 bzip2)
endif()

if (UNIX)
	if (NOT TARGET appversion)
		add_custom_target(appversion DEPENDS COMMAND "${PROJECT_SOURCE_DIR}/version/appversion.sh" "${PROJECT_SOURCE_DIR}/..")
	endif()
	add_dependencies(engine appversion)
endif()

target_include_directories(engine PRIVATE
	${PROJECT_SRC_DIR}
	${PROJECT_BZIP2_DIR}
	${PROJECT_PUBLIC_DIR}
)

target_sources(engine PRIVATE
	${COMMON_SRCS}
	${PUBLIC_SRCS}
)

target_compile_definitions(engine PRIVATE
	SWDS
	REHLDS_JIT
	REHLDS_SSE
	REHLDS_FIXES
	REHLDS_CHECKS
	REHLDS_API
	REHLDS_SELF
	REHLDS_OPT_PEDANTIC
	HAVE_OPT_STRTOOLS
	USE_BREAKPAD_HANDLER
)

if(MSVC)
	target_compile_definitions(engine PRIVATE
		WIN32
		_WINDOWS
		_WIN32
	)
	
	target_link_libraries(engine PRIVATE
		bzip2
		steam_api
		ws2_32
		winmm
	)

	set(LINK_FLAGS "${LINK_FLAGS} /LIBPATH:\"${PROJECT_SOURCE_DIR}/lib/win32\"")

else()
	target_compile_definitions(engine PRIVATE
		_LINUX
		LINUX
		_GLIBCXX_USE_CXX11_ABI=0
		_stricmp=strcasecmp
		_strnicmp=strncasecmp
		_strdup=strdup
		_unlink=unlink
		_vsnprintf=vsnprintf
		_vsnwprintf=vswprintf
	)

	target_link_libraries(engine PRIVATE
		dl
		rt
		m
		aelf32
		bzip2
		steam_api
	)

	if (USE_STATIC_LIBSTDC)
		target_compile_definitions(engine PRIVATE BUILD_STATIC_LIBSTDC)
		set(LINK_FLAGS "${LINK_FLAGS} -static-libgcc -static-libstdc++")
	endif()

	set(LINK_FLAGS "${LINK_FLAGS} -Wl,-rpath,'$ORIGIN/.' -L${PROJECT_SOURCE_DIR}/lib/linux32")
endif()

if(MSVC)
	set_target_properties(engine PROPERTIES
		OUTPUT_NAME swds
		PREFIX ""
		COMPILE_FLAGS ${COMPILE_FLAGS}
		LINK_FLAGS ${LINK_FLAGS}
	)
else()
	set_target_properties(engine PROPERTIES
		OUTPUT_NAME engine_i486
		PREFIX ""
		COMPILE_FLAGS ${COMPILE_FLAGS}
		LINK_FLAGS ${LINK_FLAGS}
		POSITION_INDEPENDENT_CODE OFF
	)
endif()