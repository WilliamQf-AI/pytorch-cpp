cmake_minimum_required(VERSION 3.28.6 FATAL_ERROR)

project(neural-style-transfer VERSION 1.0.0 LANGUAGES CXX)

set(EXECUTABLE_NAME neural-style-transfer)

add_executable(${EXECUTABLE_NAME})
target_sources(${EXECUTABLE_NAME} PRIVATE src/main.cpp
                                          src/vggnet.cpp
                                          include/vggnet.h
)
target_include_directories(${EXECUTABLE_NAME} PRIVATE include)

target_link_libraries(${EXECUTABLE_NAME} ${TORCH_LIBRARIES} image-io)

set_target_properties(${EXECUTABLE_NAME} PROPERTIES
  CXX_STANDARD 17
  CXX_STANDARD_REQUIRED YES
)

if(DOWNLOAD_DATASETS)
    add_dependencies(${EXECUTABLE_NAME} neural_style_transfer_images)
endif()

add_custom_command(TARGET ${EXECUTABLE_NAME}
                   POST_BUILD
                   COMMAND ${CMAKE_COMMAND} -E make_directory output)

if(CREATE_SCRIPTMODULES AND NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/model/vgg19_layers.pt")
    execute_process(COMMAND ${Python3_EXECUTABLE} "create_vgg19_layers_scriptmodule.py" 
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/model")
endif()

if(MSVC)
    set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_BINARY_DIR})
    set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_CURRENT_BINARY_DIR})
    include(copy_torch_dlls)
    copy_torch_dlls(${EXECUTABLE_NAME})
endif(MSVC)
