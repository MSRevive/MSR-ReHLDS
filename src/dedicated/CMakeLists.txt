add_library(steam_api SHARED IMPORTED)

target_include_directories(steam_api INTERFACE ${BASEDIR}/src/public/steam)

if (WIN32)
	set_target_properties(steam_api PROPERTIES IMPORTED_IMPLIB "${BASEDIR}/lib/public/steam_api.lib")
else()
	set_target_properties(steam_api PROPERTIES IMPORTED_IMPLIB "${BASEDIR}/lib/public/linux32/libsteam_api.so")
endif()

add_executable(dedicated)

set_common_properties(dedicated)

target_link_libraries(dedicated PRIVATE
	$<$<PLATFORM_ID:Windows>:ws2_32>
	$<$<PLATFORM_ID:Windows>:user32>
	$<$<PLATFORM_ID:Windows>:shell32>
	$<$<PLATFORM_ID:Windows>:winmm>
	$<$<PLATFORM_ID:Linux>:dl>
	$<$<PLATFORM_ID:Linux>:rt>
	steam_api
)

# Include directories
target_include_directories(dedicated PRIVATE
	${CMAKE_CURRENT_SOURCE_DIR}/src
	${BASEDIR}/src
	${BASEDIR}/src/common
	${BASEDIR}/src/engine
	${BASEDIR}/src/public
	${BASEDIR}/src/public/rehlds
	${BASEDIR}/src/pm_shared
	${BASEDIR}/src/game_shared
)

target_compile_definitions(dedicated PRIVATE
	LAUNCHER_FIXES
	_CONSOLE
	DEDICATED
)

# Set Windows subsystem to CONSOLE and use WinMain entry point
if(WIN32)
	set_target_properties(dedicated PROPERTIES
		WIN32_EXECUTABLE FALSE
	)
	target_link_options(dedicated PRIVATE
		/SUBSYSTEM:CONSOLE
		/ENTRY:WinMainCRTStartup
	)
endif()

################################################################################
# Source groups
################################################################################
set(Source_Files
	"src/conproc.cpp"
	"src/conproc.h"
	"src/dbg.cpp"
	"src/dbg.h"
	"src/dedicated_exports.cpp"
	"src/dedicated.h"
	"src/isys.h"
	"src/precompiled.cpp"
	"src/precompiled.h"
	"src/public_amalgamation.cpp"
	"src/sys_ded.cpp"
	"src/sys_ded.h"
	"src/sys_window.cpp"
	"src/vgui/vguihelpers.cpp"
	"src/vgui/vguihelpers.h"
)
source_group("Source Files" FILES ${Source_Files})

set(Common
	"../common/textconsole.cpp"
	"../common/textconsole.h"
	"../common/TextConsoleWin32.cpp"
	"../common/TextConsoleWin32.h"
	"../common/TextConsoleUnix.cpp"
	"../common/TextConsoleUnix.h"
	"../common/SteamAppStartUp.cpp"
	"../common/SteamAppStartUp.h"
	"../common/ObjectList.cpp"
	"../common/ObjectList.h"
	"../common/ObjectDictionary.cpp"
	"../common/ObjectDictionary.h"
	"../common/commandline.cpp"
	"../common/minidump.cpp"
	"../common/minidump.h"
)
source_group("Common" FILES ${Common})

set(Engine
	"../engine/mem.cpp"
	"../engine/mem.h"
)
source_group("Engine" FILES ${Engine})

set(Public
	"../public/interface.h"
	"../public/iregistry.h"
)
source_group("Public" FILES ${Public})

set(ALL_FILES
	${Source_Files}
	${Common}
	${Engine}
	${Public}
)

target_precompile_headers(dedicated PRIVATE 
	"./src/precompiled.h")
target_sources(dedicated PRIVATE ${ALL_FILES})
