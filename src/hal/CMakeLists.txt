set(${TARGET} Hal)
file(GLOB_RECURSE SRC_FILES "./*.cpp" "./*.hpp" )
add_library(${TARGET} ${LIBRARY_TYPE} ${GLOB_RECURSE})
target_include_directories(${TARGET} PUBLIC "./public/include")