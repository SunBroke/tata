#最小化版本要求
cmake_minimum_required(VERSION 3.21)

set(TARGET tata)
#定义项目名称
project(${TARGET})

#配置版本号
set(MAJOR_VERSION 1)
set(MINOR_VERSION 0)

#生成版本头文件
configure_file(version.in.h.in version.in.h @ONLY)



#设置C++20标准
set(CMAKE_CXX_STANDARD 20)
#开启module与concept特性
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fmodules -fconcepts")

#设置支持的平台架构
if(${CMAKE_SYSTEM_PROCESSOR} MATCHES "x86")
    set(PLATFORM_ARCH x86)
    add_definitions(-D_x86_)
elseif(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    set(PLATFORM_ARCH arm)
    add_definitions(-D_arm_)
elseif(${CMAKE_SYSTEM_PROCESSOR} MATCHES "x86_64")
    set(PLATFORM_ARCH x86_64)
    add_definitions(-D_x86_64_)
elseif({CMAKE_SYSTEM_PROCESSOR} MATCHES "x64")
    set(PLATFORM_ARCH x64)
    add_definitions(-D_x64_)
elseif({CMAKE_SYSTEM_PROCESSOR} MATCHES "arm64")
    set(PLATFORM_ARCH arm64)
    add_definitions(-D_arm64_)
else()
    message(FATAL_ERROR "Unsupported architecture")
endif()

#设置支持的操作系统平台
if(WIN32)
    set(PLATFORM_OS windows)
    add_definitions(-D_WIN32_)
elseif(UNIX AND NOT APPLE)
    set(PLATFORM_OS linux)
    add_definitions(-D_LINUX_ -D_UNIX_ )
elseif(APPLE)
    set(PLATFORM_OS macos)
    add_definitions(-D__APPLE__ -D__MACH__ -D__MACOS__)
else()
    message(FATAL_ERROR "Unsupported OS")
endif()

#设置库的安装路径
set(INSTALL_PATH "install" CACHE PATH "Installation directory")
set(DEPENDENCIES "")

#查找外部依赖库
find_package(OpenCV REQUIRED)
find_package(Vulkan REQUIRED)
find_package(Metal REQUIRED)
find_package(DX12 REQUIRED)

if(Not OpenCV_FOUND)
    message(WARNING “OpenCV Not Found”)
else    
    list(APPEND DEPENDENCIES ${OpenCV_LIBS})
endif()

if(Not Vulkan_FOUND)
    message(WARNING Vulkan Not Found”)
else    
    list(APPEND DEPENDENCIES ${Vulkan_LIBS})
endif()

if(Not Metal_FOUND)
    message(WARNING Metal Not Found”)
else    
    list(APPEND DEPENDENCIES ${Metal_LIBS})
endif()

if(Not DX12_FOUND)
    message(WARNING “ODX12 Not Found”)
else    
    list(APPEND DEPENDENCIES ${DX12_LIBS})
endif()

#设置spdlog和boost的路径
set(SPDLOG_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/thirdparty/spdlog/include)
set(BOOST_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/thirdparty/boost/include)

#指定头文件路径
include_directories(
    ${CMAKE_SOURCE_DIR}/src/core/public/include
    ${CMAKE_SOURCE_DIR}/src/misc/public/include
    ${CMAKE_BINARY_DIR}
    ${OpenCV_INCLUDE_DIRS}
    ${Vulkan_INCLUDE_DIRS}
    ${Metal_INCLUDE_DIRS}
    ${DX12_INCLUDE_DIRS}
    ${SPDLOG_INCLUDE_DIR}
    ${BOOST_INCLUDE_DIR}
)
#源文件目录
add_subdirectory(src)
#配置编译选项

#配置生成可执行文件或库
set(BUILD_SHARED_LIBS ON)
option(BUILD_APP "Build Executable" ON)
option(BUILD_LIB "Build Library" OFF)


#option 只在当前文件中有作用，使用set可以做缓存，其他文件也共用
#option(BUILD_SHARED_LIBS "Build shared/dynamic libraries instead of static libraries" ON)
if(BUILD_SHARED_LIBS)
    set(LIBRARY_TYPE SHARED)
else()
    set(LIBRARY_TYPE STATIC)
endif()

#配置编译类型
if(CMAKE_BUILD_TYPE MATCHES "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -g -D_DEBUG")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -DNDEBUG")
endif()

# 遍历src文件夹
#file(GLOB_RECURSE SRC_FILES "src/*.cpp" "src/*.hpp")

# 遍历src文件夹，并搜索一级子目录下的.cpp和.hpp文件
file(GLOB SRC_FILES "src/*.cpp" "src/*.hpp" )

add_subdirectory(src/core)
add_subdirectory(src/misc)
add_subdirectory(src/hal)
add_subdirectory(src/math)


list(APPEND DEPENDENCIES Core Misc Hal Math)
message("Dependencies :" ${DEPENDENCIES})
if(BUILD_APP)
    add_executable(${TARGET} ${SRC_FILES})
    target_link_libraries(${TAEGET} ${DEPENDCIES})
endif()

if(BUILD_LIB)
add_library(${TARGET} ${LIBRARY_TYPE} ${SRC_FILES})
target_link_libraries(${TAEGET} ${DEPENDCIES})
endif()

#可选安装模块
option(BUILD_INSTALLER "Build an installer for MyProject" OFF)
if(BUILD_INSTALLER)
    install(TARGETS Core Misc
        RUNTIME DESTINATION ${INSTALL_PATH}/bin
        LIBRARY DESTINATION ${INSTALL_PATH}/lib
        ARCHIVE DESTINATION ${INSTALL_PATH}/lib/static
    )
    install(DIRECTORY ${CMAKE_SOURCE_DIR}/thirdparty/spdlog DESTINATION ${INSTALL_PATH})
    install(DIRECTORY ${CMAKE_SOURCE_DIR}/thirdparty/boost DESTINATION ${INSTALL_PATH})
    install(DIRECTORY ${OpenCV_DIR}/lib DESTINATION ${INSTALL_PATH})
    install(DIRECTORY ${Vulkan_LIB_DIR}/lib DESTINATION ${INSTALL_PATH})
    install(DIRECTORY ${Metal_LIB_DIR}/lib DESTINATION ${INSTALL_PATH})
    install(DIRECTORY ${DX12_LIB_DIR}/lib DESTINATION ${INSTALL_PATH})
endif()