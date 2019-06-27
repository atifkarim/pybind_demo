cmake_minimum_required(VERSION 3.1)
project(pybind_demo)

set(CMAKE_CXX_STANDARD 14)
set(LIB_GEN_PATH ${PROJECT_SOURCE_DIR}/scripts CACHE STRING "Where generated library will be placed")
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif() 

find_package(PythonLibs 3 REQUIRED)

message(STATUS "Submodule update")
execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                RESULT_VARIABLE GIT_SUBMOD_RESULT)
if(NOT GIT_SUBMOD_RESULT EQUAL "0")
    message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
endif()

if(NOT EXISTS "${CMAKE_SOURCE_DIR}/pybind11/CMakeLists.txt")
    message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()

add_subdirectory(pybind11)
pybind11_add_module(${PROJECT_NAME} src/${PROJECT_NAME}.cpp src/bindings.cpp)
target_include_directories(${PROJECT_NAME} PRIVATE ${PYTHON_INCLUDE_DIRS} include)
set_target_properties(${PROJECT_NAME}
    PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY ${LIB_GEN_PATH}
)
set_source_files_properties(src/${PROJECT_NAME}.cpp 
    PROPERTIES COMPILE_FLAGS -Wall -Wextra -Weffc++ -pedantic -Werror
)
