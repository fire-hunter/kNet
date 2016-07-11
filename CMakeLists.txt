# Copyright 2010 Jukka Jylänki

#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at

#      http://www.apache.org/licenses/LICENSE-2.0

#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.

# Modified by Lasse Oorni and Yao Wei Tjong for Urho3D

cmake_minimum_required(VERSION 3.5)
project(kNet VERSION 2.7.0)
file (GLOB kNetSourceFiles src/*.cpp)
file (GLOB kNetHeaderFiles include/*.h include/kNet/*.h include/kNet/*.inl)

if (WIN32)
   file (GLOB kNetWin32SourceFiles src/win32/*.cpp)
   file (GLOB kNetWin32HeaderFiles include/kNet/win32/*.h)

   if (USE_BOOST)
      list (REMOVE_ITEM kNetWin32SourceFiles ${CMAKE_CURRENT_SOURCE_DIR}/src/win32/W32Thread.cpp)
   endif ()

   list (APPEND kNetSourceFiles ${kNetWin32SourceFiles})
   list (APPEND kNetHeaderFiles ${kNetWin32HeaderFiles})

   add_definitions (-D_WINSOCKAPI_)
   add_definitions (-D_CRT_SECURE_NO_WARNINGS)
   add_definitions (-DKNET_MEMORY_LEAK_CHECK)
elseif (UNIX)
   file (GLOB kNetUnixSourceFiles src/unix/*.cpp)
   file (GLOB kNetUnixHeaderFiles include/*.h include/kNet/*.h include/kNet/unix/*.h)

   if (USE_BOOST)
      list (REMOVE_ITEM kNetUnixSourceFiles ${CMAKE_CURRENT_SOURCE_DIR}/src/unix/UnixThread.cpp)
   endif ()

   list (APPEND kNetSourceFiles ${kNetUnixSourceFiles})
   list (APPEND kNetHeaderFiles ${kNetUnixHeaderFiles})

   # Urho3D: KNET_UNIX definition removed
endif ()

# Urho3D: set DEBUG_CPP_NAME only on Windows, Xcode project file can not be opened if this is included
if (WIN32)
    foreach (srcFile ${kNetSourceFiles})
        get_filename_component (baseName ${srcFile} NAME)
        set_source_files_properties (${srcFile} PROPERTIES COMPILE_FLAGS "-DDEBUG_CPP_NAME=\"\\\"${baseName}\"\\\"")
    endforeach ()
endif ()
set (INCLUDE_DIRS include)

include_directories("${INCLUDE_DIRS}")
add_library(${PROJECT_NAME} "${kNetSourceFiles}")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(include_install_dir "include")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
  )

configure_package_config_file(
  "cmake/Config.cmake.in"
  "${project_config}"
  INSTALL_DESTINATION "${config_install_dir}"
  )

install(
  TARGETS ${PROJECT_NAME}
  EXPORT "${targets_export_name}"
  LIBRARY DESTINATION "lib"
  ARCHIVE DESTINATION "lib"
  RUNTIME DESTINATION "bin"
  INCLUDES DESTINATION "${include_install_dir}/${PROJECT_NAME}"
  )

install (
  FILES ${kNetHeaderFiles} ${kNetUnixHeaderFiles} ${kNetWin32HeaderFiles}
  DESTINATION ${include_install_dir}/${PROJECT_NAME})

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
  )
