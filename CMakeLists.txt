cmake_minimum_required(VERSION 3.15)

project(EmImEntGL VERSION 1.0)

add_library(project_compiler_flags INTERFACE)
target_compile_features(project_compiler_flags INTERFACE cxx_std_20)

set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC,Emscripten>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")

target_compile_options(project_compiler_flags INTERFACE
    "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
    "$<${msvc_cxx}:$<BUILD_INTERFACE:-W4>>"
)

set(source_files
    src/main.cpp
    src/BaseApp.cpp
    src/BaseApp.h
    src/Demo.cpp
    src/Demo.h
    src/GLFWLib.h)

if (EMSCRIPTEN)
    message(STATUS "This is an Emscripten build.")
    list(APPEND source_files
        src/WebApp.cpp
        src/WebApp.h)
    target_link_options(project_compiler_flags INTERFACE
        -sUSE_WEBGL2=1
        -sUSE_GLFW=3
        -sALLOW_MEMORY_GROWTH=1)
    set(CMAKE_EXECUTABLE_SUFFIX ".html")
    configure_file(scripts/server.sh.in server.sh)
else()
    list(APPEND source_files
        src/WindowApp.cpp
        src/WindowApp.h
        3rdParty/glad/src/gles2.c)
    set(glfw_dir 3rdParty/glfw-3.4)
    set(glfw_include_dir ${glfw_dir}/include)
    add_subdirectory(${glfw_dir})
endif()

set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT EmImEntGL)

add_executable(EmImEntGL ${source_files})

if (NOT EMSCRIPTEN)
    target_include_directories(EmImEntGL PRIVATE 3rdParty/glad/include)
    target_include_directories(EmImEntGL PRIVATE ${glfw_include_dir})
    target_link_libraries(EmImEntGL PRIVATE glfw ${GLFW_LIBRARIES})
endif()

add_subdirectory(3rdParty/imgui-1.92.1)
target_include_directories(EmImEntGL PRIVATE 3rdParty/imgui-1.92.1)
target_link_libraries(EmImEntGL PRIVATE ImGui)

target_compile_definitions(EmImEntGL PRIVATE
    "$<$<COMPILE_LANG_AND_ID:CXX,Emscripten>:EMSCRIPTEN_BUILD>"
)

target_link_libraries(EmImEntGL PUBLIC project_compiler_flags)

install(TARGETS EmImEntGL DESTINATION bin)
