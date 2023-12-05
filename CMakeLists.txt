#--------------------------------------------------------------------------------
# Copyright 2009-2023 CLO Virtual Fashion. All rights reserved.
# Sample CMakeLists.txt file for ExportPlugin

#--------------------------------------------------------------------------------
# Minimum CMake version
cmake_minimum_required(VERSION 3.20)

#--------------------------------------------------------------------------------
# Target definitions
message("\n>> Copyright 2009-2023 CLO Virtual Fashion. All rights reserved.")
message("\n>> Configuring Sample Project for ExportPlugin\n")

project(
    ExportPlugin
    DESCRIPTION "Sample Export Plugin for CLO"
    VERSION 7.1.194
    LANGUAGES CXX C
)

add_library(${PROJECT_NAME} SHARED "")

#--------------------------------------------------------------------------------
# Compiler settings
set_target_properties(${PROJECT_NAME} PROPERTIES
    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
)

#--------------------------------------------------------------------------------
# Runtime library options
if (MSVC)
    set_target_properties(${PROJECT_NAME} PROPERTIES
        MSVC_RUNTIME_LIBRARY "MultiThreadedDLL"
    )
endif()

#--------------------------------------------------------------------------------
# Preprocessor definitions
if(APPLE)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        CLO_API_INTERFACE_STATIC_BUILD
    )
elseif(MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        UNICODE
        _WINDOWS
        _USRDLL
        BOMAPIDLL_EXPORTS
    )
endif()

#--------------------------------------------------------------------------------
# macOS specific settings
if(APPLE)
    # Deployment target
    if ("${CMAKE_OSX_DEPLOYMENT_TARGET}" STREQUAL "")
        set(CMAKE_OSX_DEPLOYMENT_TARGET 11.0) # Big Sur
    endif()
    # Target architectures
    if ("${CMAKE_OSX_ARCHITECTURES}" STREQUAL "")
        execute_process(COMMAND uname -m COMMAND tr -d '\n' OUTPUT_VARIABLE osx_architecture)
        set(CMAKE_OSX_ARCHITECTURES ${osx_architecture})
    elseif( NOT ((${CMAKE_OSX_ARCHITECTURES} MATCHES "x86_64") OR (${CMAKE_OSX_ARCHITECTURES} MATCHES "arm64")) )
        message(FATAL_ERROR "Target architecture not supported: ${CMAKE_OSX_ARCHITECTURES}")
    endif()
    message("-- Target macOS architecture (${PROJECT_NAME}): ${CMAKE_OSX_ARCHITECTURES}")
endif()

#--------------------------------------------------------------------------------
# Source
if (MSVC)
    target_sources(${PROJECT_NAME} PRIVATE
        dllmain.cpp
    )
endif()

target_sources(${PROJECT_NAME} PRIVATE
    ExportPlugin.cpp
    ExportPlugin.h
)

#--------------------------------------------------------------------------------
# CLOAPIInterface
find_library(CLO_API_INTERFACE NAMES CLOAPIInterface PATHS ../../CLOAPIInterface/Lib REQUIRED)

if (MSVC)
    find_file(CLO_API_INTERFACE_DLL NAMES CLOAPIInterface.dll PATHS ../../CLOAPIInterface/Lib REQUIRED)
    list(APPEND STATIC_LIBRARIES ${CLO_API_INTERFACE})
    list(APPEND SHARED_LIBRARIES ${CLO_API_INTERFACE_DLL})
elseif (APPLE)
    list(APPEND SHARED_LIBRARIES ${CLO_API_INTERFACE})
endif()

target_include_directories(${PROJECT_NAME} PRIVATE
    ../../CLOAPIInterface
)

#--------------------------------------------------------------------------------
# Linking
target_link_libraries(${PROJECT_NAME} PRIVATE ${Qt5_MODULES})
if (MSVC)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        ${STATIC_LIBRARIES}
    )
elseif(APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        ${SHARED_LIBRARIES}
    )
endif()

#--------------------------------------------------------------------------------
# Post build events

# [macOS] Ensure to adjust install names to use the same libraries as the host app
if (APPLE)
    add_custom_command(
        TARGET ${PROJECT_NAME}
        POST_BUILD
        COMMAND ${CMAKE_INSTALL_NAME_TOOL} -change
            \"@rpath/libCLOAPIInterface.dylib\"
            \"@executable_path/../Frameworks/libCLOAPIInterface.dylib\"
            "lib${PROJECT_NAME}.dylib"
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    )
endif()