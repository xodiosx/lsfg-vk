cmake_minimum_required(VERSION 3.10)
project(lsfg-vk LANGUAGES CXX)
include(GNUInstallDirs)

# ============================================================
# Build options
# ============================================================
option(LSFGVK_BUILD_VK_LAYER     "Build the Vulkan layer"            ON)
option(LSFGVK_BUILD_UI           "Build the user interface"         OFF)
option(LSFGVK_BUILD_CLI          "Build the command line interface"  OFF)
option(LSFGVK_INSTALL_DEVELOP    "Install development libraries and headers"      OFF)
option(LSFGVK_INSTALL_XDG_FILES  "Install the application icon and desktop files" OFF)
set(LSFGVK_LAYER_LIBRARY_PATH liblsfg-vk-layer.so CACHE STRING "Change where Vulkan searches for the layer library")
option(LSFGVK_TESTING_RENDERDOC  "Enable RenderDoc integration for testing purposes" OFF)

# ============================================================
# Android / Termux‑bionic target
# ============================================================
# When using the NDK toolchain, CMAKE_SYSTEM_NAME is "Android".
# We'll use that to automatically enable Android Vulkan support.
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
    set(ANDROID_BUILD ON)
else()
    set(ANDROID_BUILD OFF)
endif()

# But you can also force it manually:
option(ANDROID_BUILD_FORCE "Force Android Vulkan platform support" OFF)
if(ANDROID_BUILD_FORCE)
    set(ANDROID_BUILD ON)
endif()

# ============================================================
# Global compile settings
# ============================================================
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_SKIP_RPATH ON)

if(ANDROID_BUILD)
    # Let volk expose Android-specific extensions (required for VK_KHR_android_surface, etc.)
    # Although we use FD path, this ensures the Vulkan loader sees the Android platform.
    set(VOLK_STATIC_DEFINES VK_USE_PLATFORM_ANDROID_KHR CACHE STRING "" FORCE)
    add_compile_definitions(VK_USE_PLATFORM_ANDROID_KHR)
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        message(STATUS "Building with clang-specific diagnostics")
        set(CMAKE_CXX_CLANG_TIDY clang-tidy)
        add_compile_options(
            -Weverything
            -Wno-pre-c++20-compat-pedantic
            -Wno-c++98-compat-pedantic
            -Wno-switch-default
            -Wno-missing-designated-field-initializers
            -Wno-shadow
            -Wno-cast-function-type-strict
            -Wno-padded
        )
    endif()
endif()

if(LSFGVK_TESTING_RENDERDOC)
    add_compile_definitions(LSFGVK_TESTING_RENDERDOC)
endif()

# ============================================================
# Sub‑directories (these exist in PancakeTAS repo)
# ============================================================
add_subdirectory(lsfg-vk-common)
add_subdirectory(lsfg-vk-backend)
if(LSFGVK_BUILD_VK_LAYER)
    add_subdirectory(lsfg-vk-layer)
endif()
if(LSFGVK_BUILD_UI)
    add_subdirectory(lsfg-vk-ui)
endif()
if(LSFGVK_BUILD_CLI)
    add_subdirectory(lsfg-vk-cli)
endif()

# ============================================================
# Install (only needed for packaging)
# ============================================================
if(LSFGVK_BUILD_VK_LAYER)
    install(FILES "${CMAKE_BINARY_DIR}/liblsfg-vk-layer.so"
        DESTINATION lib)
    install(FILES "${CMAKE_SOURCE_DIR}/VkLayer_LS_frame_generation.json"
        DESTINATION share/vulkan/implicit_layer.d)
endif()
