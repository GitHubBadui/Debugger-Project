cmake_minimum_required(VERSION 3.16)

project(debugger LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Supports C++ files at the project root and under src/.
file(GLOB_RECURSE DEBUGGER_SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cxx"
)
file(GLOB ROOT_SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/*.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
    "${CMAKE_CURRENT_SOURCE_DIR}/*.cxx"
)
list(APPEND DEBUGGER_SOURCES ${ROOT_SOURCES})

# linenoise.h only declares these functions; its C implementation must be linked.
# Add other vendored locations here if your checkout uses a different layout.
set(LINENOISE_CANDIDATES
    "${CMAKE_CURRENT_SOURCE_DIR}/linenoise/linenoise.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/third_party/linenoise/linenoise.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/external/linenoise/linenoise.c"
    "${CMAKE_CURRENT_SOURCE_DIR}/vendor/linenoise/linenoise.c"
)
foreach(LINENOISE_FILE IN LISTS LINENOISE_CANDIDATES)
    if(EXISTS "${LINENOISE_FILE}")
        list(APPEND DEBUGGER_SOURCES "${LINENOISE_FILE}")
        get_filename_component(LINENOISE_INCLUDE_DIR "${LINENOISE_FILE}" DIRECTORY)
        break()
    endif()
endforeach()

if(NOT LINENOISE_INCLUDE_DIR AND
   NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/linenoise.c" AND
   NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/src/linenoise.c")
    message(FATAL_ERROR "linenoise.c not found. Add its source file to the project and list its path in CMakeLists.txt.")
endif()

if(NOT DEBUGGER_SOURCES)
    message(FATAL_ERROR "No C++ sources found at the project root or under src/.")
endif()

add_executable(debugger ${DEBUGGER_SOURCES})
target_include_directories(debugger PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
    "${CMAKE_CURRENT_SOURCE_DIR}/src"
    "${LINENOISE_INCLUDE_DIR}"
)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(debugger PRIVATE -Wall -Wextra -Wpedantic)
endif()
