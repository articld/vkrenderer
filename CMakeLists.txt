cmake_minimum_required(VERSION 3.29)

project(vkrenderer LANGUAGES CXX)

option(ENABLE_CPP20_MODULE
        "Enable the experimental Vulkan C++20 module"
        OFF)

if(ENABLE_CPP20_MODULE)
    set(CMAKE_CXX_SCAN_FOR_MODULES ON)
endif()

# The tutorial uses its own FindVulkan.cmake.
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMake")

find_package(Vulkan REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)
find_package(stb REQUIRED)

# Provide Vulkan::cppm in either mode.
if(ENABLE_CPP20_MODULE)
    add_library(VulkanCppModule)
    add_library(Vulkan::cppm ALIAS VulkanCppModule)

    target_compile_definitions(VulkanCppModule PUBLIC
            VULKAN_HPP_DISPATCH_LOADER_DYNAMIC=1
            VULKAN_HPP_NO_STRUCT_CONSTRUCTORS=1
    )

    target_include_directories(VulkanCppModule PUBLIC
            "${Vulkan_INCLUDE_DIR}"
    )

    target_link_libraries(VulkanCppModule PUBLIC Vulkan::Vulkan)

    set_target_properties(VulkanCppModule PROPERTIES
            CXX_STANDARD 20
            CXX_STANDARD_REQUIRED YES
    )

    target_sources(VulkanCppModule
            PUBLIC
            FILE_SET cxx_modules TYPE CXX_MODULES
            BASE_DIRS "${Vulkan_INCLUDE_DIR}"
            FILES "${Vulkan_INCLUDE_DIR}/vulkan/vulkan.cppm"
    )
else()
    # Header-based Vulkan-Hpp mode; this is the default.
    add_library(VulkanCppModule INTERFACE)
    add_library(Vulkan::cppm ALIAS VulkanCppModule)

    target_link_libraries(VulkanCppModule INTERFACE Vulkan::Vulkan)

    target_compile_definitions(VulkanCppModule INTERFACE
            VULKAN_HPP_DISPATCH_LOADER_DYNAMIC=1
            VULKAN_HPP_NO_STRUCT_CONSTRUCTORS=1
    )
endif()

add_executable(vkrenderer
        src/main.cpp
)

set_target_properties(vkrenderer PROPERTIES
        CXX_STANDARD 20
        CXX_STANDARD_REQUIRED YES
        CXX_EXTENSIONS NO
)

target_link_libraries(vkrenderer PRIVATE
        Vulkan::cppm
        glfw
        glm::glm
)

target_include_directories(vkrenderer PRIVATE
        ${stb_INCLUDE_DIRS}
)

target_compile_definitions(vkrenderer PRIVATE
        VULKAN_HPP_HANDLE_ERROR_OUT_OF_DATE_AS_SUCCESS
)

if(ENABLE_CPP20_MODULE)
    target_compile_definitions(vkrenderer PRIVATE
            USE_CPP20_MODULES=1
    )
endif()
