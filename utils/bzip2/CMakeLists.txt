add_library(bzip2 STATIC)

# Make bzlib.h available to targets that link against bzip2
target_include_directories(bzip2 PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

################################################################################
# Source groups
################################################################################
set(Header_Files
    "bzlib.h"
    "bzlib_private.h"
)
source_group("Header Files" FILES ${Header_Files})

set(Src_Files
    "blocksort.c"
    "bzlib.c"
    "compress.c"
    "crctable.c"
    "decompress.c"
    "huffman.c"
    "randtable.c"
)
source_group("Source Files" FILES ${Src_Files})

set(ALL_FILES
    ${Header_Files}
    ${Src_Files}
)

target_sources(bzip2 PRIVATE ${ALL_FILES})