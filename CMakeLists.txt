cmake_minimum_required(VERSION 3.16)

project(debugger LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Supports main.cpp at the project root and C++ files under src/.
file(GLOB_RECURSE DEBUGGER_SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cxx"
)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
    list(APPEND DEBUGGER_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
endif()

if(NOT DEBUGGER_SOURCES)
    message(FATAL_ERROR "No C++ sources found. Add main.cpp or files under src/.")
endif()

add_executable(debugger ${DEBUGGER_SOURCES})
target_include_directories(debugger PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
    "${CMAKE_CURRENT_SOURCE_DIR}/src"
)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(debugger PRIVATE -Wall -Wextra -Wpedantic)
endif()
