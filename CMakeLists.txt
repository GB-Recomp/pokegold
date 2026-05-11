cmake_minimum_required(VERSION 3.16)
project(pokegold C CXX)

include(CheckIPOSupported)
include(FetchContent)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-O0 -g)
    else()
        add_compile_options(-O3)
    endif()
endif()

# ---------------------------------------------------------------------------
# gb-recompiled runtime
# When consumed by an umbrella compilation (e.g. GB-Recomp/pgbcomp), the
# parent is expected to provide `gbrt`. We detect that and skip the fetch.
# ---------------------------------------------------------------------------
if(NOT TARGET gbrt)
    set(GBRT_REF "main" CACHE STRING "git ref of GB-Recomp/gb-recompiled to use")
    FetchContent_Declare(gb_recompiled
        GIT_REPOSITORY https://github.com/GB-Recomp/gb-recompiled.git
        GIT_TAG ${GBRT_REF}
        GIT_SHALLOW TRUE
    )
    FetchContent_GetProperties(gb_recompiled)
    if(NOT gb_recompiled_POPULATED)
        FetchContent_Populate(gb_recompiled)
    endif()
    add_subdirectory(${gb_recompiled_SOURCE_DIR}/runtime ${CMAKE_BINARY_DIR}/_gbrt_build)
endif()

# ---------------------------------------------------------------------------
# Cart sources
# ---------------------------------------------------------------------------
set(POKEGOLD_SOURCES
    pokegold_main.c
    pokegold.c
    pokegold_rom.c
    pokegold_dispatch_chunk_0.c
    pokegold_dispatch_chunk_1.c
    pokegold_dispatch_chunk_2.c
    pokegold_dispatch_chunk_3.c
    pokegold_funcs_0.c
    pokegold_funcs_1.c
    pokegold_funcs_2.c
    pokegold_funcs_3.c
    pokegold_funcs_4.c
    pokegold_funcs_5.c
    pokegold_funcs_6.c
    pokegold_funcs_7.c
    pokegold_funcs_8.c
    pokegold_funcs_9.c
    pokegold_funcs_10.c
    pokegold_funcs_11.c
    pokegold_funcs_12.c
    pokegold_funcs_13.c
    pokegold_funcs_14.c
    pokegold_funcs_15.c
    pokegold_funcs_16.c
    pokegold_funcs_17.c
    pokegold_funcs_18.c
    pokegold_funcs_19.c
    pokegold_funcs_20.c
    pokegold_funcs_21.c
    pokegold_funcs_22.c
    pokegold_funcs_23.c
    pokegold_funcs_24.c
    pokegold_funcs_25.c
    pokegold_funcs_26.c
    pokegold_funcs_27.c
    pokegold_funcs_28.c
    pokegold_funcs_29.c
    pokegold_funcs_30.c
    pokegold_funcs_31.c
    pokegold_funcs_32.c
    pokegold_funcs_33.c
    pokegold_funcs_34.c
    pokegold_funcs_35.c
    pokegold_funcs_36.c
    pokegold_funcs_37.c
    pokegold_funcs_38.c
    pokegold_funcs_39.c
    pokegold_funcs_40.c
    pokegold_funcs_41.c
    pokegold_funcs_42.c
    pokegold_funcs_43.c
    pokegold_funcs_44.c
    pokegold_funcs_45.c
    pokegold_funcs_46.c
    pokegold_funcs_47.c
    pokegold_funcs_48.c
    pokegold_funcs_49.c
    pokegold_funcs_50.c
    pokegold_funcs_51.c
    pokegold_funcs_52.c
)

set(GBRECOMP_GENERATED_OPT_LEVEL "3" CACHE STRING "Optimization level for generated ROM source files")
set_property(CACHE GBRECOMP_GENERATED_OPT_LEVEL PROPERTY STRINGS 0 1 2 3)
set_source_files_properties(${POKEGOLD_SOURCES} PROPERTIES COMPILE_OPTIONS "-O${GBRECOMP_GENERATED_OPT_LEVEL}")

# ---------------------------------------------------------------------------
# pokegold_cart: static library so umbrella compilations can link us in.
# Exposes `pokegold_main(argc, argv)` for the launcher's g_games[].
# ---------------------------------------------------------------------------
add_library(pokegold_cart STATIC ${POKEGOLD_SOURCES})
target_include_directories(pokegold_cart PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(pokegold_cart PUBLIC gbrt)

# ---------------------------------------------------------------------------
# Standalone launcher executable. Only built when this is the top-level
# CMake project.
# ---------------------------------------------------------------------------
if(CMAKE_PROJECT_NAME STREQUAL "pokegold")
    add_executable(pokegold_launcher launcher_main.cpp)
    target_link_libraries(pokegold_launcher PRIVATE pokegold_cart gbrt)
    set_target_properties(pokegold_launcher PROPERTIES OUTPUT_NAME pokegold)
endif()
