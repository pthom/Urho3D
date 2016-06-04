#
# Copyright (c) 2008-2016 the Urho3D project.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#

# Check existence of stdint.h for LibCpuId
include (CheckIncludeFiles)
check_include_files (stdint.h HAVE_STDINT_H)
if (HAVE_STDINT_H)
    add_definitions (-DHAVE_STDINT_H)
endif ()

# Setup RPATH settings
if (URHO3D_LIB_TYPE STREQUAL SHARED AND NOT WIN32 AND NOT ANDROID AND NOT IOS AND NOT WEB)
    # Add RPATH entries when building
    set (CMAKE_SKIP_BUILD_RPATH FALSE)
    # But don't set them yet in the build tree
    set (CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
    if (APPLE)
        if (CMAKE_VERSION VERSION_GREATER 2.8.12 OR CMAKE_VERSION VERSION_EQUAL 2.8.12)
            set (CMAKE_MACOSX_RPATH TRUE)
        endif ()
        set (ORIGIN @loader_path)
    else ()
        set (ORIGIN $ORIGIN)
    endif ()
    # When installing/packaging set the first RPATH entry to the library location relative to the executable
    set (CMAKE_INSTALL_RPATH ${ORIGIN}/../lib${LIB_SUFFIX}/${PATH_SUFFIX})    # The library location is based on SDK install destination
    # The second entry to the library location relative to the executable in the tool directory
    list (APPEND CMAKE_INSTALL_RPATH ${ORIGIN}/../../lib${LIB_SUFFIX}/${PATH_SUFFIX})    # The tools are installed one directory further down from normal executable
    # The last entry to the install destination of the library, if the destination location is not in the system default search path
    list (FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX} isSystemDir)
    if (isSystemDir STREQUAL -1)
        list (APPEND CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_PREFIX}/${DEST_LIBRARY_DIR})
    endif ()
endif ()

# Add targets

# NOTE: FreeType is satisfied by hunter
# NOTE: rapidjson is satisified by hunter
## #################################################################
## Dependencies - FreeType
## #################################################################
hunter_add_package(freetype)
find_package(freetype CONFIG REQUIRED)
## #################################################################
## Dependencies - RapidJSON
## #################################################################
hunter_add_package(rapidjson)
find_package(rapidjson REQUIRED)
## #################################################################
## Dependencies - JO
## #################################################################
hunter_add_package(JO)
find_package(JO CONFIG REQUIRED)
## #################################################################
## Dependencies - LZ4
## #################################################################
hunter_add_package(LZ4)
find_package(LZ4 CONFIG REQUIRED)
## #################################################################
## Dependencies - PugiXml
## #################################################################
hunter_add_package(PugiXml)
find_package(PugiXml CONFIG REQUIRED)
## #################################################################
## Dependencies - StanHull
## #################################################################
hunter_add_package(StanHull)
find_package(StanHull CONFIG REQUIRED)
## #################################################################
## Dependencies - STB
## #################################################################
hunter_add_package(STB)
find_package(STB CONFIG REQUIRED)

# no pkg-config:
include_directories(${RAPIDJSON_INCLUDE_DIR})

get_cmake_property(_variableNames VARIABLES)
foreach (_variableName ${_variableNames})
  message(STATUS "VARIABLE: ${_variableName}=${${_variableName}}")
endforeach()

set(HUNTER_THIRD_PARTY_DEPS freetype::freetype JO::JO LZ4::LZ4 PugiXml::PugiXml StanHull::StanHull STB::STB)

foreach (TARGET SDL)
    add_subdirectory (ThirdParty/${TARGET})
endforeach ()

if (URHO3D_ANGELSCRIPT)
    add_subdirectory (ThirdParty/AngelScript)
endif ()

if (URHO3D_LUA)
    add_subdirectory (ThirdParty/Lua${JIT})
    add_subdirectory (ThirdParty/toluapp/src/lib)
endif ()

if (URHO3D_NETWORK)
  add_subdirectory (ThirdParty/Civetweb)

  ## #################################################################
  ## Dependencies - kNet
  ## #################################################################
  #add_subdirectory (ThirdParty/kNet)
  hunter_add_package(kNet)
  find_package(kNet CONFIG REQUIRED)
  list(APPEND HUNTER_THIRD_PARTY_DEPS kNet::kNet)
endif ()

if (URHO3D_DATABASE_ODBC)
    add_subdirectory (ThirdParty/nanodbc)
endif ()
if (URHO3D_DATABASE_SQLITE)
  ## #################################################################
  ## Dependencies - SQLite
  ## #################################################################
  #add_subdirectory (ThirdParty/SQLite)
  message("USE SQLITE")
  hunter_add_package(SQLite)
  find_package(SQLite CONFIG REQUIRED)
  list(APPEND HUNTER_THIRD_PARTY_DEPS sqlite::sqlite)
endif ()

if (URHO3D_NAVIGATION)
  message(HELLO)
  return()
  ## #################################################################
  ## Dependencies - Box2D
  ## #################################################################
  #add_subdirectory (ThirdParty/Detour)
  hunter_add_package(Detour)
  find_package(Detour CONFIG REQUIRED)
  list(APPEND HUNTER_THIRD_PARTY_DEPS Detour::Detour)
  
  add_subdirectory (ThirdParty/DetourCrowd)
  add_subdirectory (ThirdParty/DetourTileCache)
  add_subdirectory (ThirdParty/Recast)
endif ()

if (URHO3D_URHO2D)
  ## #################################################################
  ## Dependencies - Box2D
  ## #################################################################
  #add_subdirectory (ThirdParty/Box2D)
  hunter_add_package(Box2D)
  find_package(Box2D CONFIG REQUIRED)
  list(APPEND HUNTER_THIRD_PARTY_DEPS Box2D::Box2D)
  
  if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/ThirdParty/spine)
    add_subdirectory (ThirdParty/spine)
    # Add definition for Spine
    add_definitions (-DURHO3D_SPINE)
  endif ()
endif ()

if (URHO3D_PHYSICS)
  ## #################################################################
  ## Dependencies - Bullet
  ## #################################################################
  #add_subdirectory (ThirdParty/Bullet)
  hunter_add_package(Bullet)
  find_package(Bullet CONFIG REQUIRED)
  list(APPEND HUNTER_THIRD_PARTY_DEPS Bullet::Bullet)
endif ()

if (NOT IOS AND NOT ANDROID AND NOT RPI AND NOT WEB)
  if (URHO3D_OPENGL)

    ## #################################################################
    ## Dependencies - GLEW
    ## #################################################################
    #add_subdirectory (ThirdParty/GLEW)
    hunter_add_package(GLEW)
    find_package(GLEW CONFIG REQUIRED)
    list(APPEND HUNTER_THIRD_PARTY_DEPS GLEW::GLEW)
    
  elseif (NOT URHO3D_D3D11)

    ## #################################################################
    ## Dependencies - MojoShader
    ## #################################################################
    #add_subdirectory (ThirdParty/MojoShader)      
    hunter_add_package(MojoShader)
    find_package(MojoShader CONFIG REQUIRED)
    list(APPEND HUNTER_THIRD_PARTY_DEPS MojoShader::MojoShader)

  endif ()

  ## #################################################################
  ## Dependencies - LibCpuId
  ## #################################################################
  #add_subdirectory (ThirdParty/LibCpuId)
  hunter_add_package(LibCpuId)
  find_package(LibCpuId CONFIG REQUIRED)
  list(APPEND HUNTER_THIRD_PARTY_DEPS LibCpuId::LibCpuId)
  
endif ()

# Urho3D game engine library
add_subdirectory (Urho3D)

# Urho3D Clang tools
if (URHO3D_CLANG_TOOLS)
    add_subdirectory (Clang-Tools)
endif ()

# Urho3D tools
add_subdirectory (Tools)

# Urho3D samples
# Samples are built on iOS platform too when enabled. Currently there is no available mechanism to package each sample apps into individual *.apk
# for Android platform, so we deploy all samples into one apk and use a simple list view activity launcher to select the sample to run
if (URHO3D_SAMPLES)
    add_subdirectory (Samples)
endif ()

# Urho3D extras
if (URHO3D_EXTRAS)
    add_subdirectory (Extras)
endif ()
