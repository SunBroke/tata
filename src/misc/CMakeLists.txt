set(${TARGET} Core)
file(GLOB_RECURSE SRC_FILES "./*.cpp" "./*.hpp" )
add_library(${TARGET} ${LIBRARY_TYPE} ${GLOB_RECURSE})