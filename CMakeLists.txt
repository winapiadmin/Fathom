cmake_minimum_required(VERSION 3.10)
project(fathom C CXX)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

set(FATHOM_SRC
    src/tbprobe.c
    src/tbchess.c
)

add_library(fathom SHARED ${FATHOM_SRC})
target_include_directories(fathom PUBLIC src)
set_property(TARGET fathom PROPERTY POSITION_INDEPENDENT_CODE ON)

# Compiler-specific options
# - Treat clang-cl as MSVC-like (it uses MSVC frontend flags)
if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
    target_compile_options(fathom PRIVATE -O2 -Wall -Wshadow)
elseif (CMAKE_C_COMPILER_ID STREQUAL "Clang")
    if (DEFINED CMAKE_C_COMPILER_FRONTEND_VARIANT AND CMAKE_C_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
        # This is clang-cl (Clang with MSVC frontend)
        target_compile_options(fathom PRIVATE /O2 /W4)
    else()
        # regular clang (Unix-style)
        target_compile_options(fathom PRIVATE -O2 -Wall -Wshadow)
    endif()
elseif (CMAKE_C_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(fathom PRIVATE /O2 /W4)
else()
    # Fallback — be conservative
    target_compile_options(fathom PRIVATE $<$<COMPILE_LANGUAGE:C>:-O2>)
endif()

# Install
include(GNUInstallDirs)
install(TARGETS fathom
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(FILES
    src/stdendian.h
    src/tbconfig.h
    src/tbprobe.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
