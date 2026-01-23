add_library(filesystem SHARED)

set_common_properties(filesystem)

target_link_libraries(filesystem PRIVATE
	$<$<PLATFORM_ID:Linux>:dl>
)

# Include directories
target_include_directories(filesystem PRIVATE
	${CMAKE_CURRENT_SOURCE_DIR}/src
	${BASEDIR}/src
	${BASEDIR}/src/common
	${BASEDIR}/src/public
	${BASEDIR}/src/public/rehlds
	${BASEDIR}/src/pm_shared
	${BASEDIR}/src/game_shared
)

target_compile_definitions(filesystem PRIVATE
	FILESYSTEM_STDIO_EXPORTS
)

# NOTE: Don't use IPO or LTO with '--wrap' may make this feature ineffective
set(WRAP_FUNCS_LIST
	"scandir" "opendir" "freopen" "fopen" "fopen64" "open" "open64" "creat"
	"access" "stat" "lstat" "__xstat" "__lxstat" "__xstat64" "__lxstat64"
	"chmod" "chown" "lchown" "unlink" "symlink" "link" "mknod" "mount"
	"mkfifo" "rename" "utime" "utimes" "mkdir" "rmdir"
)

foreach(f ${WRAP_FUNCS_LIST})
	set(LINK_FLAGS "${LINK_FLAGS} -Wl,-wrap,${f}")
endforeach()

set_target_properties(filesystem PROPERTIES OUTPUT_NAME "FileSystem_Stdio")
if (UNIX)
	set_target_properties(filesystem PROPERTIES OUTPUT_NAME "filesystem_stdio")
	set_target_properties(filesystem PROPERTIES LINK_FLAGS ${LINK_FLAGS})
endif(UNIX)

################################################################################
# Source groups
################################################################################
set(Src_Files
	"src/BaseFileSystem.cpp"
	"src/BaseFileSystem.h"
	"src/filesystem_helpers.cpp"
	"src/filesystem_helpers.h"
	"src/FileSystem_Stdio.cpp"
	"src/FileSystem_Stdio.h"
	"src/linux_support.cpp"
	"src/linux_support.h"
	"src/pathmatch.cpp"
	"src/pathmatch_casefolding.h"
	"src/precompiled.cpp"
	"src/precompiled.h"
	"src/public_amalgamation.cpp"
)
source_group("Source Files" FILES ${Src_Files})

set(ALL_FILES
	${Source_Files}
)

target_precompile_headers(filesystem PRIVATE 
	"./src/precompiled.h")
target_sources(filesystem PRIVATE ${ALL_FILES})
