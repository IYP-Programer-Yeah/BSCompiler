project(BSCompiler)

# Add the location of LLVMConfig.cmake to CMake search paths (so that
# find_package can locate it)
list(APPEND CMAKE_MODULE_PATH "${LLVM_BUILD_DIRECTORY}/lib/cmake/llvm/")
list(APPEND CMAKE_PREFIX_PATH "${LLVM_BUILD_DIRECTORY}/lib/cmake/llvm/")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")

message(${LLVM_BUILD_DIRECTORY})

find_package(LLVM REQUIRED CONFIG)

set(LLVM_ENABLE_PLUGINS ON)
include(AddLLVM)

include_directories(${LLVM_INCLUDE_DIRS})
message(${LLVM_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})
message(${LLVM_DEFINITIONS})
link_directories(${LLVM_LIBRARY_DIRS})

file(GLOB_RECURSE CPP_FILES ${REPO_ROOT}/${PROJECT_NAME}/Source/*.cpp)

#add_library(${PROJECT_NAME} SHARED ${CPP_FILES})

add_llvm_library( ${PROJECT_NAME} MODULE ${CPP_FILES})

target_link_libraries(${PROJECT_NAME} LLVMPasses)
