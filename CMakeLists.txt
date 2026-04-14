cmake_minimum_required(VERSION 3.10)
project(ocgcore)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_LUA "Build with bundled Lua" ON)
option(BUILD_SHARED_LIBS "Build shared library" ON)
option(ARCH_32 "Build 32-bit library" OFF)

set(LUA_DIR "${CMAKE_SOURCE_DIR}/lua" CACHE PATH "Path to Lua source directory")

if(BUILD_LUA)
    if(NOT EXISTS "${LUA_DIR}/src lua.h")
        message(FATAL_ERROR "Lua source not found at ${LUA_DIR}/src. Set LUA_DIR or extract lua source.")
    endif()
    add_subdirectory(${LUA_DIR} lua EXCLUDE_FROM_ALL)
    set(LUA_INCLUDE_DIR "${LUA_DIR}/src")
    set(LUA_LIBRARIES lua)
endif()

include_directories(${LUA_INCLUDE_DIR})

set(OCGCORE_SOURCES
    card.cpp card.h card_data.h
    common.h
    duel.cpp duel.h
    effect.cpp effect.h effectset.h
    field.cpp field.h
    group.cpp group.h
    interpreter.cpp interpreter.h
    libcard.cpp
    libdebug.cpp
    libduel.cpp
    libeffect.cpp
    libgroup.cpp
    mem.cpp
    mtrandom.h
    ocgapi.cpp ocgapi.h
    operations.cpp
    playerop.cpp
    scriptlib.cpp scriptlib.h
    sort.h
)

if(BUILD_SHARED_LIBS)
    add_library(ocgcore SHARED ${OCGCORE_SOURCES})
else()
    add_library(ocgcore STATIC ${OCGCORE_SOURCES})
endif()

target_link_libraries(ocgcore ${LUA_LIBRARIES})

if(UNIX AND NOT APPLE)
    target_link_options(ocgcore PRIVATE -static-libgcc -static-libstdc++)
    target_compile_definitions(ocgcore PRIVATE LUA_USE_LINUX)
endif()

if(MSVC)
    target_compile_definitions(ocgcore PRIVATE _CRT_SECURE_NO_WARNINGS)
    target_compile_options(ocgcore PRIVATE /utf-8)
endif()
