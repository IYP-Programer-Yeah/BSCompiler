set(LLVM_REPO_ROOT ${REPO_ROOT}/Dependencies/llvm-project/llvm)
set(LLVM_BUILD_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_GENERATOR}/${CMAKE_BUILD_TYPE})
string(REPLACE " " "-" LLVM_BUILD_DIRECTORY ${LLVM_BUILD_DIRECTORY})
set(LLVM_BUILD_DIRECTORY ${LLVM_BUILD_DIRECTORY} PARENT_SCOPE)
set(TEMP_PRE_HASH_FILE ${LLVM_BUILD_DIRECTORY}/pre-source-hash.tmp)
set(SOURCE_FILES_HASH_FILE ${LLVM_BUILD_DIRECTORY}/source-hash.tmp)
set(LLVM_TARGETS LLVMPasses LLVMSupport opt)

file(WRITE ${TEMP_PRE_HASH_FILE} "${LLVM_TARGETS}")
file(GLOB_RECURSE SOURCE_FILES FOLLOW_SYMLINKS LIST_DIRECTORIES false ${LLVM_REPO_ROOT}/*)
foreach(SOURCE_FILE ${SOURCE_FILES})
    file(MD5 ${SOURCE_FILE} SOURCE_FILE_HASH)
    file(APPEND ${TEMP_PRE_HASH_FILE} ${SOURCE_FILE})
    file(APPEND ${TEMP_PRE_HASH_FILE} ${SOURCE_FILE_HASH})
endforeach()

file(MD5 ${TEMP_PRE_HASH_FILE} SOURCE_FILES_HASH)
message("Current hash is: ${SOURCE_FILES_HASH}")

set(CACHE_HASH "DEFAULT_VALUE")
if(EXISTS "${SOURCE_FILES_HASH_FILE}")
    file(READ ${SOURCE_FILES_HASH_FILE} CACHE_HASH)
    message("Found previous hash: ${CACHE_HASH}")
endif()

if(NOT "${CMAKE_BUILD_TYPE}")
    set(LLVM_BUILD_OPTIONS "--config Release" "--config Debug")
else()
    set(LLVM_CONFIGURATION_OPTION "-DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}")
endif()

if ((NOT (EXISTS "${SOURCE_FILES_HASH_FILE}")) OR (NOT (EXISTS "${LLVM_BUILD_DIRECTORY}")) OR (NOT (${CACHE_HASH} STREQUAL ${SOURCE_FILES_HASH})))
    message("Building LLVM! This is gonna take some time!")
    file(WRITE ${SOURCE_FILES_HASH_FILE} "DEFAULT_VALUE")
    execute_process(COMMAND ${CMAKE_COMMAND} -E make_directory ${LLVM_BUILD_DIRECTORY}
        COMMAND ${CMAKE_COMMAND} -S ${LLVM_REPO_ROOT} -B ${LLVM_BUILD_DIRECTORY} -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_OPTIMIZED_TABLEGEN=ON ${LLVM_CONFIGURATION_OPTION} -G ${CMAKE_GENERATOR}
        WORKING_DIRECTORY ${LLVM_BUILD_DIRECTORY}
        RESULTS_VARIABLE LLVM_CONFIGURATION_RESULT)

    if ("${LLVM_CONFIGURATION_RESULT}" STREQUAL "0;0")
        message("Starting LLVM build.")
        if (MSVC)
            set(LLVM_BUILD_CONFIGS Release Debug)
            foreach(LLVM_BUILD_CONFIG ${LLVM_BUILD_CONFIGS})
                execute_process(COMMAND ${CMAKE_COMMAND} --build ${LLVM_BUILD_DIRECTORY} --config ${LLVM_BUILD_CONFIG} --target ${LLVM_TARGETS}
                    WORKING_DIRECTORY ${LLVM_BUILD_DIRECTORY}
                    RESULTS_VARIABLE LLVM_BUILD_RESULT)
            endforeach()
        else()
            execute_process(COMMAND cmake --build ${LLVM_BUILD_DIRECTORY} --target ${LLVM_TARGETS}
                WORKING_DIRECTORY ${LLVM_BUILD_DIRECTORY}
                RESULTS_VARIABLE LLVM_BUILD_RESULT)
        endif()

    else()
        message("LLVM configuration failed.")
    endif()

    if ("${LLVM_BUILD_RESULT}" STREQUAL "0")
        message("Build task passed with code: ${LLVM_BUILD_RESULT}")
        message("Writing cache")
        file(WRITE ${SOURCE_FILES_HASH_FILE} ${SOURCE_FILES_HASH})
    endif()
endif()
