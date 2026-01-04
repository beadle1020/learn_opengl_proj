cmake_minimum_required(VERSION 3.17)
project(learn_opengl_proj)

set(CMAKE_CXX_STANDARD 17)

link_directories(${CMAKE_SOURCE_DIR}/lib)
include_directories(${CMAKE_SOURCE_DIR}/includes)

configure_file(configuration/root_directory.h.in configuration/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/configuration)

# define copy resource function
function(copy_resource target path)
    file(GLOB RESOURCES
        "${path}/*.vs"
        "${path}/*.fs"
        "${path}/*.gs"
    )
    foreach(RESOURCE ${RESOURCES})
        if(WIN32)
            message(STATUS "RESOURCE file ${RESOURCE}")
            add_custom_command(TARGET ${target} POST_BUILD COMMAND
                ${CMAKE_COMMAND} -E copy ${RESOURCE} $<TARGET_FILE_DIR:${target}>)
        endif(WIN32)
    endforeach(RESOURCE)
endfunction(copy_resource)

function(each_file root_dir cb_func)
#    message(STATUS "root dir: ${root_dir}")
    file(GLOB files RELATIVE ${CMAKE_SOURCE_DIR} ${root_dir}/*)
    foreach(file ${files})
#        message(STATUS "file: ${file}")
        cmake_language(CALL ${cb_func} ${file} ${ARGN})
    endforeach()
endfunction()

function(process_section sub_dir)
#    message(STATUS "process_section: ${sub_dir}")

    if(NOT IS_DIRECTORY "${CMAKE_SOURCE_DIR}/${sub_dir}")
#        message(STATUS "not dir: ${sub_dir}")
        return()
    endif()

    string(REGEX REPLACE "^.*src/([0-9]+)\\.[^/]+/([^/]+)$"
        "\\1.\\2" exec_name ${sub_dir})
    message(STATUS "exec_name: ${exec_name}")

    set(current_dir ${CMAKE_SOURCE_DIR}/${sub_dir})

    set(main_file ${current_dir}/main.cpp)
    set(link_file ${current_dir}/link.txt)
    if(EXISTS ${link_file})
        file(READ ${link_file} link_path)
        get_filename_component(main_file ${current_dir}/${link_path}/main.cpp ABSOLUTE)
#        message(STATUS "link main_file: ${main_file}")
    endif()

    set(sources
        ${CMAKE_SOURCE_DIR}/src/glad.c
        ${CMAKE_SOURCE_DIR}/src/stb_image.cpp
        ${main_file}
    )
    add_executable(${exec_name} ${sources})
    # set target properties
    set_target_properties(${exec_name} PROPERTIES
        OUTPUT_NAME "run"
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/${exec_name}
    )
    copy_resource(${exec_name} ${sub_dir})
    target_link_libraries(${exec_name} libglfw3.a)
endfunction()

function(process_src_dir sub_dir)
#    message(STATUS "process_src_dir: ${sub_dir}")

    if(NOT IS_DIRECTORY "${CMAKE_SOURCE_DIR}/${sub_dir}")
#        message(STATUS "not dir: ${sub_dir}")
        return()
    endif()

    each_file(${sub_dir} process_section)
endfunction()

# iterate each "src/*" sub folder
each_file(src process_src_dir)