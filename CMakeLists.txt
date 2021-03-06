cmake_minimum_required(VERSION 3.4.3)

project(servise_mask CXX)

find_package(InferenceEngine REQUIRED)
find_package(OpenCV REQUIRED)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

add_subdirectory(3rdparty/gtest)

add_definitions(-DDATA_FOLDER="${PROJECT_SOURCE_DIR}/data")

file(GLOB modules "modules/*")
foreach(module ${modules})
    get_filename_component(module_name ${module} NAME)
    string(REGEX REPLACE "[0-9]+_" "" module_name ${module_name})

    # Build library
    file(GLOB SOURCES "${module}/src/*.cpp")
    if (SOURCES)
        add_library(module_${module_name} STATIC ${SOURCES})
        target_include_directories(module_${module_name} PUBLIC "${module}/include")
        target_link_libraries(module_${module_name} PUBLIC ${OpenCV_LIBRARIES} ${InferenceEngine_LIBRARIES})
    endif()

    # Compile test
    file(GLOB TEST_SOURCES "${module}/test/*.cpp")
    if (TEST_SOURCES)
        add_executable(test_${module_name} "${TEST_SOURCES}")
        target_include_directories(test_${module_name} PRIVATE "${PROJECT_SOURCE_DIR}/3rdparty/")
        target_link_libraries(test_${module_name} PRIVATE gtest module_${module_name})
        if (UNIX)
            target_link_libraries(test_${module_name} PRIVATE pthread)
        endif()
    endif()

    file(GLOB PYTEST_FILE "${module}/test_${module_name}.py")
    if (PYTEST_FILE)
        execute_process(
            COMMAND ${CMAKE_COMMAND} -E copy
                ${PYTEST_FILE}
                ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/test_${module_name})
    endif()
endforeach()
