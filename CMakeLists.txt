cmake_minimum_required(VERSION 3.5)

set(ARI_VERSION_MAJOR 1)
set(ARI_VERSION_MINOR 0)
set(ARI_VERSION_MICRO 3)
set(ARI_VERSION ${ARI_VERSION_MAJOR}.${ARI_VERSION_MINOR}.${ARI_VERSION_MICRO})
set(VERSION ${ARI_VERSION})

set(PACKAGE aribb24)

project(aribb24 VERSION ${ARI_VERSION})

option(BUILD_SHARED_LIBS "Build shared library." ON)
option(BUILD_STATIC_LIBS "Build static library." ON)

find_package(PNG)

if (PNG_FOUND)
    add_definitions(-DHAVE_PNG)
endif()

if (MSVC)
    add_compile_options(/we4013)
endif()

include(CheckSymbolExists)
include(GNUInstallDirs)

check_symbol_exists(vasprintf "stdio.h" HAVE_VASPRINTF)

if (HAVE_VASPRINTF)
    add_definitions(-DHAVE_VASPRINTF)
endif()

set(LIB_OBJS src/aribb24.c src/aribb24_private.h
src/decoder.c src/decoder_private.h src/parser.c
src/parser_private.h src/md5.c src/md5.h src/drcs.c
src/drcs.h src/convtable.h src/decoder_macro.h)

if (MSVC)
    list(APPEND LIB_OBJS src/asprintf.c src/asprintf.h)
endif()

set(PUBLIC_HEADERS src/aribb24/decoder.h src/aribb24/parser.h
src/aribb24/bits.h src/aribb24/aribb24.h)

if (BUILD_SHARED_LIBS)
    add_definitions(-DARIBB24_DLL)
    add_library(libaribb24 SHARED ${LIB_OBJS})
    set_target_properties(libaribb24 PROPERTIES OUTPUT_NAME aribb24)
    set_target_properties(libaribb24 PROPERTIES PUBLIC_HEADER "${PUBLIC_HEADERS}")
    list(APPEND TARGETS libaribb24)
endif()
if (BUILD_STATIC_LIBS)
    add_library(libaribb24-static STATIC ${LIB_OBJS})
    if (NOT MSVC OR (MSVC AND NOT BUILD_SHARED_LIBS))
        set_target_properties(libaribb24-static PROPERTIES OUTPUT_NAME aribb24)
    else()
        set_target_properties(libaribb24-static PROPERTIES OUTPUT_NAME aribb24-static)
    endif()
    set_target_properties(libaribb24-static PROPERTIES PUBLIC_HEADER "${PUBLIC_HEADERS}")
    list(APPEND TARGETS libaribb24-static)
endif()

if (PNG_FOUND)
    include_directories(${PNG_INCLUDE_DIRS})
endif()

foreach(TAR ${TARGETS})
    if (PNG_FOUND)
        target_link_libraries(${TAR} PNG::PNG)
    endif()
endforeach()

install(TARGETS ${TARGETS}
PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/aribb24")
