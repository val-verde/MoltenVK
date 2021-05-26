cmake_minimum_required(VERSION 3.22)

if(NOT DEFINED MoltenVK_VERSION)
    find_package(Git)

    execute_process(
        COMMAND ${GIT_EXECUTABLE} --git-dir ${CMAKE_SOURCE_DIR}/.git describe --abbrev=0 --tags
        OUTPUT_VARIABLE MoltenVK_VERSION
    )
    STRING(STRIP ${MoltenVK_VERSION} MoltenVK_VERSION)
    STRING(REPLACE "v" "" MoltenVK_VERSION ${MoltenVK_VERSION})
endif()

message(STATUS "Configuring Molten VK ${MoltenVK_VERSION}.")
project(MoltenVK VERSION ${MoltenVK_VERSION} LANGUAGES CXX OBJC OBJCXX)

option(BUILD_SHARED_LIBRARY "Build MoltenVK as a shared library (default = TRUE)." TRUE)

set(DISABLED_WARNINGS "-Wno-deprecated-declarations -Wno-tautological-pointer-compare")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${DISABLED_WARNINGS}")
set(CMAKE_OBJC_FLAGS "${CMAKE_OBJC_FLAGS} ${DISABLED_WARNINGS}")
set(CMAKE_OBJCXX_FLAGS "${CMAKE_OBJCXX_FLAGS} ${DISABLED_WARNINGS}")

find_package(Threads REQUIRED)
find_package(glslang REQUIRED)
find_package(SPIRV-Tools REQUIRED)
find_package(VulkanHeaders REQUIRED)
find_library(SPIRV_CROSS_CORE_LIBRARY spirv-cross-core REQUIRED)
find_library(SPIRV_CROSS_GLSL_LIBRARY spirv-cross-glsl REQUIRED)
find_library(SPIRV_CROSS_HLSL_LIBRARY spirv-cross-hlsl REQUIRED)
find_library(SPIRV_CROSS_MSL_LIBRARY spirv-cross-msl REQUIRED)
find_library(SPIRV_CROSS_REFLECT_LIBRARY spirv-cross-reflect REQUIRED)
find_library(SPIRV_CROSS_CPP_LIBRARY spirv-cross-cpp REQUIRED)
find_library(SPIRV_CROSS_C_LIBRARY spirv-cross-c REQUIRED)
find_library(SPIRV_TOOLS_OPT_LIBRARY SPIRV-Tools-opt REQUIRED)
find_library(CoreGraphics_LIBRARY CoreGraphics REQUIRED)
find_library(Foundation_LIBRARY Foundation REQUIRED)
find_library(IOSurface_LIBRARY IOSurface REQUIRED)
find_library(Metal_LIBRARY Metal REQUIRED)
find_library(QuartzCore_LIBRARY QuartzCore REQUIRED)

if(IOS OR CMAKE_SYSTEM_NAME STREQUAL "tvOS")
    find_library(AppKit_OR_UIKit_LIBRARY UIKit)
else()
    find_library(AppKit_OR_UIKit_LIBRARY AppKit)
    find_library(IOKit_LIBRARY IOKit)
endif()

file(
    GLOB_RECURSE MoltenVK_PUBLIC_HEADERS
    MoltenVK/MoltenVK/API/*.h
)

file(
    GLOB_RECURSE MoltenVK_SOURCES
    Common/*.cpp Common/*.m Common/*.mm
    MoltenVK/*.cpp MoltenVK/*.m MoltenVK/*.mm
    MoltenVKShaderConverter/*.cpp MoltenVKShaderConverter/*.m MoltenVKShaderConverter/*.mm
)

set(
    MoltenVK_INCLUDES
    ${CMAKE_SOURCE_DIR}
    Common
    MoltenVK/MoltenVK/API
    MoltenVK/MoltenVK/Commands
    MoltenVK/MoltenVK/GPUObjects
    MoltenVK/MoltenVK/Layers
    MoltenVK/MoltenVK/OS
    MoltenVK/MoltenVK/Utility
    MoltenVK/MoltenVK/Vulkan
    MoltenVKShaderConverter
    MoltenVKShaderConverter/MoltenVKShaderConverter
)

if(BUILD_SHARED_LIBRARY)
    set(MoltenVK_LIBRARY_TYPE SHARED)
else()
    set(MoltenVK_LIBRARY_TYPE STATIC)
endif()

add_library(
    MoltenVK ${MoltenVK_LIBRARY_TYPE}
    ${MoltenVK_PUBLIC_HEADERS}
    ${MoltenVK_SOURCES}
)

target_include_directories(
    MoltenVK
    PRIVATE
        ${MoltenVK_INCLUDES}
        Vulkan::Headers
)

target_compile_definitions(
    MoltenVK
    PRIVATE MVK_GIT_REV="${MoltenVK_VERSION}"
)

get_filename_component(
    SPIRV_TOOLS_OPT_LIBRARY_DIR
    ${SPIRV_TOOLS_OPT_LIBRARY}
    DIRECTORY
)

target_link_directories(
    MoltenVK PUBLIC
    ${SPIRV_TOOLS_OPT_LIBRARY_DIR}
)

target_link_libraries(
    MoltenVK PUBLIC
    SPIRV-Tools
    glslang::glslang
    glslang::SPIRV
    ${AppKit_OR_UIKit_LIBRARY}
    ${CoreGraphics_LIBRARY}
    ${Foundation_LIBRARY}
    ${IOKit_LIBRARY}
    ${IOSurface_LIBRARY}
    ${Metal_LIBRARY}
    ${QuartzCore_LIBRARY}
    ${SPIRV_CROSS_C_LIBRARY}
    ${SPIRV_CROSS_CORE_LIBRARY}
    ${SPIRV_CROSS_CPP_LIBRARY}
    ${SPIRV_CROSS_GLSL_LIBRARY}
    ${SPIRV_CROSS_HLSL_LIBRARY}
    ${SPIRV_CROSS_MSL_LIBRARY}
    ${SPIRV_CROSS_REFLECT_LIBRARY}
)

set_target_properties(
    MoltenVK PROPERTIES
    OUTPUT_NAME MoltenVK
    FRAMEWORK TRUE
    FRAMEWORK_VERSION ${MoltenVK_VERSION}
    PUBLIC_HEADER "${MoltenVK_PUBLIC_HEADERS}"
    MACOSX_BUNDLE_BUNDLE_NAME "MoltenVK"
    MACOSX_BUNDLE_BUNDLE_VERSION ${MoltenVK_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${MoltenVK_VERSION}
    MACOSX_FRAMEWORK_IDENTIFIER "com.moltenvk.MoltenVK"
)

install(
    TARGETS MoltenVK
    FRAMEWORK DESTINATION .
    LIBRARY DESTINATION lib
)
