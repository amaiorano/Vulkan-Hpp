# Copyright(c) 2015-2018, NVIDIA CORPORATION. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

cmake_minimum_required(VERSION 3.2)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

project(VulkanHppGenerator)

find_program(CLANG_FORMAT_EXECUTABLE NAMES clang-format)

if(CLANG_FORMAT_EXECUTABLE)
  # get the clang-format version string
  execute_process(COMMAND ${CLANG_FORMAT_EXECUTABLE} "--version" OUTPUT_VARIABLE clangFormatVersion)
  # filter out the actual version
  string(REGEX MATCH [0123456789.]+ clangFormatVersion "${clangFormatVersion}")
  # we need at least version 7.0.0 !
  if (clangFormatVersion VERSION_LESS 7.0.0)
	message(WARNING " Found too old clang-format version <" ${clangFormatVersion} ">, we need version 7 and up to nicely format vulkan.hpp and vulkan_raii.hpp")
  else ()
	message(STATUS " Found clang-format version <" ${clangFormatVersion} ">.")
	add_definitions(-DCLANG_FORMAT_EXECUTABLE="${CLANG_FORMAT_EXECUTABLE}")
	if (clangFormatVersion VERSION_LESS 11.0.0)
		message(STATUS " Using .clang-format version 7." )
		file(READ ".clang-format_7" clangFormat)
	else ()
		message(STATUS " Using .clang-format version 11." )
		file(READ ".clang-format_11" clangFormat)
	endif ()
	file(WRITE ".clang-format" ${clangFormat})
  endif()
else()
  message(WARNING " Could not find clang-format. Generated vulkan.hpp and vulkan_raii.hpp will not be nicely formatted.")
endif()

if (NOT DEFINED CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 11)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)

message("CMAKE_CXX_STANDARD = <${CMAKE_CXX_STANDARD}>")

if (NOT DEFINED VulkanRegistry_DIR)
  if (DEFINED VULKAN_HPP_VULKAN_HEADERS_SRC_DIR)
    set(VulkanRegistry_DIR "${VULKAN_HPP_VULKAN_HEADERS_SRC_DIR}/registry")
  else()
    set(VulkanRegistry_DIR "${CMAKE_CURRENT_SOURCE_DIR}/Vulkan-Headers/registry")
  endif()
endif()
file(TO_NATIVE_PATH ${VulkanRegistry_DIR}/vk.xml vk_spec)
string(REPLACE "\\" "\\\\" vk_spec ${vk_spec})
add_definitions(-DVK_SPEC="${vk_spec}")

if (NOT DEFINED VulkanHeaders_INCLUDE_DIR)
  if (DEFINED VULKAN_HPP_PATH)
    set(VulkanHeaders_INCLUDE_DIR ${VULKAN_HPP_PATH})
  else()
    set(VulkanHeaders_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
  endif()
endif()
file(TO_NATIVE_PATH ${VulkanHeaders_INCLUDE_DIR}/vulkan/vulkan.hpp vulkan_hpp)
string(REPLACE "\\" "\\\\" vulkan_hpp ${vulkan_hpp})
file(TO_NATIVE_PATH ${VulkanHeaders_INCLUDE_DIR}/vulkan/vulkan_enums.hpp vulkan_enums_hpp)
string(REPLACE "\\" "\\\\" vulkan_enums_hpp ${vulkan_enums_hpp})
file(TO_NATIVE_PATH ${VulkanHeaders_INCLUDE_DIR}/vulkan/vulkan_funcs.hpp vulkan_funcs_hpp)
string(REPLACE "\\" "\\\\" vulkan_funcs_hpp ${vulkan_funcs_hpp})
file(TO_NATIVE_PATH ${VulkanHeaders_INCLUDE_DIR}/vulkan/vulkan_handles.hpp vulkan_handles_hpp)
string(REPLACE "\\" "\\\\" vulkan_handles_hpp ${vulkan_handles_hpp})
file(TO_NATIVE_PATH ${VulkanHeaders_INCLUDE_DIR}/vulkan/vulkan_structs.hpp vulkan_structs_hpp)
string(REPLACE "\\" "\\\\" vulkan_structs_hpp ${vulkan_structs_hpp})
file(TO_NATIVE_PATH ${VulkanHeaders_INCLUDE_DIR}/vulkan/vulkan_hash.hpp vulkan_hash_hpp)
string(REPLACE "\\" "\\\\" vulkan_hash_hpp ${vulkan_hash_hpp})
file(TO_NATIVE_PATH ${VulkanHeaders_INCLUDE_DIR}/vulkan/vulkan_raii.hpp vulkan_raii_hpp)
string(REPLACE "\\" "\\\\" vulkan_raii_hpp ${vulkan_raii_hpp})
add_definitions(-DVULKAN_HPP_FILE="${vulkan_hpp}"
                -DVULKAN_ENUMS_HPP_FILE="${vulkan_enums_hpp}"
                -DVULKAN_FUNCS_HPP_FILE="${vulkan_funcs_hpp}"
                -DVULKAN_HANDLES_HPP_FILE="${vulkan_handles_hpp}"
                -DVULKAN_STRUCTS_HPP_FILE="${vulkan_structs_hpp}"
                -DVULKAN_HASH_HPP_FILE="${vulkan_hash_hpp}"
                -DVULKAN_RAII_HPP_FILE="${vulkan_raii_hpp}")
include_directories(${VulkanHeaders_INCLUDE_DIR})

set(HEADERS
  VulkanHppGenerator.hpp
)

set(SOURCES
  VulkanHppGenerator.cpp
)

if (NOT DEFINED VULKAN_HPP_TINYXML2_SRC_DIR)
  set(VULKAN_HPP_TINYXML2_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/tinyxml2")
endif()

set(TINYXML2_SOURCES
  ${VULKAN_HPP_TINYXML2_SRC_DIR}/tinyxml2.cpp
)

set(TINYXML2_HEADERS
  ${VULKAN_HPP_TINYXML2_SRC_DIR}/tinyxml2.h
)

source_group(headers FILES ${HEADERS})
source_group(sources FILES ${SOURCES})

source_group(TinyXML2\\headers FILES ${TINYXML2_HEADERS})
source_group(TinyXML2\\sources FILES ${TINYXML2_SOURCES})

add_executable(VulkanHppGenerator
  ${HEADERS}
  ${SOURCES}
  ${TINYXML2_SOURCES}
  ${TINYXML2_HEADERS}
)

set_property(TARGET VulkanHppGenerator PROPERTY CXX_STANDARD 17)

if(MSVC)
  target_compile_options(VulkanHppGenerator PRIVATE /W4 /WX)
  if (MSVC_VER GREATER_EQUAL 1910)
   target_compile_options(VulkanHppGenerator PRIVATE /permissive-)
  endif()
else(MSVC)
  target_compile_options(VulkanHppGenerator PRIVATE -Wall -Wextra -pedantic -Werror)
endif(MSVC)

target_include_directories(VulkanHppGenerator PRIVATE ${VULKAN_HPP_TINYXML2_SRC_DIR})

option (VULKAN_HPP_RUN_GENERATOR "Run the HPP generator" OFF)
if (VULKAN_HPP_RUN_GENERATOR)
  add_custom_command(
    COMMAND VulkanHppGenerator
    OUTPUT "${vulkan_hpp}"
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}"
    COMMENT "run VulkanHppGenerator"
    DEPENDS VulkanHppGenerator "${vk_spec}")

  add_custom_target(build_vulkan_hpp ALL
    DEPENDS "${vulkan_hpp}" "${vk_spec}")
endif()

option (SAMPLES_BUILD "Build samples" OFF)
if (SAMPLES_BUILD)
  # external libraries
  add_subdirectory(glm)
  set(GLFW_BUILD_EXAMPLES OFF)
  set(GLFW_BUILD_TESTS OFF)
  add_subdirectory(glfw)
  add_subdirectory(glslang)
  # samples
  add_subdirectory(samples)
  add_subdirectory(RAII_Samples)
endif ()

option (TESTS_BUILD "Build tests" OFF)
if (TESTS_BUILD)
  add_subdirectory(tests)
endif ()

if (${VULKAN_HPP_INSTALL})
  install(FILES ${vulkan_hpp} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/vulkan)
endif()
