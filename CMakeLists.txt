cmake_minimum_required(VERSION 3.20)
project(miniredis LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(ENABLE_ASAN "Build with AddressSanitizer" OFF)
option(ENABLE_TSAN "Build with ThreadSanitizer" OFF)

if(ENABLE_ASAN AND ENABLE_TSAN)
  message(FATAL_ERROR "ASan and TSan cannot be enabled together")
endif()

if(ENABLE_ASAN)
  add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer -g)
  add_link_options(-fsanitize=address,undefined)
endif()

if(ENABLE_TSAN)
  add_compile_options(-fsanitize=thread -g)
  add_link_options(-fsanitize=thread)
endif()

add_library(miniredis_core
  src/protocol/resp_parser.cpp
)
target_include_directories(miniredis_core PUBLIC src)
target_compile_options(miniredis_core PRIVATE -Wall -Wextra -Wpedantic)

enable_testing()
add_subdirectory(tests)