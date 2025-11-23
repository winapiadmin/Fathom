cmake_minimum_required(VERSION 3.10)
project(fathom_launcher C CXX)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Common sources for all launchers
set(LAUNCHER_SRC
    fathom.c
    ../tbprobe.c
)

# Global compile definitions & flags
add_compile_definitions(TB_NO_THREADS)
include(CheckCCompilerFlag)

# Compiler-specific handling
if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
    add_compile_options(-O2 -Wall -Wshadow)
elseif (CMAKE_C_COMPILER_ID STREQUAL "Clang")
    if (DEFINED CMAKE_C_COMPILER_FRONTEND_VARIANT AND CMAKE_C_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
        add_compile_options(/O2 /W4)
    else()
        add_compile_options(-O2 -Wall -Wshadow)
    endif()
elseif (CMAKE_C_COMPILER_ID STREQUAL "MSVC")
    add_compile_options(/O2 /W4)
endif()

# ____________________________________________________________
# 1) Native build: Linux, macOS, Windows (MSVC or MinGW)
# ____________________________________________________________

set(BIN_NAME "fathom")

if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    add_executable(${BIN_NAME}.linux ${LAUNCHER_SRC})
elseif (CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    add_executable(${BIN_NAME}.macosx ${LAUNCHER_SRC})
elseif (WIN32)
    add_executable(${BIN_NAME}.exe ${LAUNCHER_SRC})
endif()

# ____________________________________________________________
# 2) Optional cross-build: MinGW Windows executable on Linux
#    This replicates your "fathom.windows" target.
# ____________________________________________________________

option(BUILD_MINGW "Build Windows executable using MinGW-w64" OFF)

if (BUILD_MINGW)
    find_program(MINGW_C x86_64-w64-mingw32-gcc)
    find_program(MINGW_STRIP x86_64-w64-mingw32-strip)

    if (MINGW_C)
        add_executable(fathom.windows ${LAUNCHER_SRC})
        set_target_properties(fathom.windows PROPERTIES
            OUTPUT_NAME "fathom"
            SUFFIX ".exe"
            C_COMPILER ${MINGW_C}
        )
        add_custom_command(TARGET fathom.windows POST_BUILD
            COMMAND ${MINGW_STRIP} $<TARGET_FILE:fathom.windows>
        )
    endif()
endif()

# ____________________________________________________________
# Strip native binaries after build
# ____________________________________________________________

find_program(STRIP strip)

if (STRIP)
    if (TARGET fathom.linux)
        add_custom_command(TARGET fathom.linux POST_BUILD
            COMMAND ${STRIP} $<TARGET_FILE:fathom.linux>)
    endif()

    if (TARGET fathom.macosx)
        add_custom_command(TARGET fathom.macosx POST_BUILD
            COMMAND ${STRIP} $<TARGET_FILE:fathom.macosx>)
    endif()

    if (TARGET fathom.exe)
        add_custom_command(TARGET fathom.exe POST_BUILD
            COMMAND ${STRIP} $<TARGET_FILE:fathom.exe>
            VERBATIM)
    endif()
endif()

# ____________________________________________________________
# 3) Packaging
# ____________________________________________________________

set(INSTALL_DIR Fathom-1.0)
set(PACKAGE_NAME Fathom-1.0.zip)

add_custom_target(release ALL
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${INSTALL_DIR}
    COMMAND ${CMAKE_COMMAND} -E make_directory ${INSTALL_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<$<TARGET_EXISTS:fathom.linux>:$<TARGET_FILE:fathom.linux>>
        ${INSTALL_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<$<TARGET_EXISTS:fathom.macosx>:$<TARGET_FILE:fathom.macosx>>
        ${INSTALL_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<$<TARGET_EXISTS:fathom.exe>:$<TARGET_FILE:fathom.exe>>
        ${INSTALL_DIR}
    COMMAND ${CMAKE_COMMAND} -E tar cf ${PACKAGE_NAME} --format=zip ${INSTALL_DIR}
)
