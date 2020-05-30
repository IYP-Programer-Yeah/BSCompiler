cmake_minimum_required (VERSION 3.16)

set(REPO_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../..)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(llvm)
add_subdirectory(BSCompiler)
