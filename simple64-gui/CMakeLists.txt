cmake_minimum_required(VERSION 3.20)

project(simple64-gui LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(CheckIPOSupported)
check_ipo_supported(RESULT ENABLE_IPO)
if(ENABLE_IPO)
    message("Interprocedural optimizations enabled")
endif(ENABLE_IPO)

if(ENABLE_IPO AND (CMAKE_BUILD_TYPE STREQUAL "Release"))
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
endif()

include_directories(../mupen64plus-core/src/api)

add_definitions(-DQT_DISABLE_DEPRECATED_BEFORE=0x0602ff -DNETPLAY_VER=10)

if (DEFINED ENV{SIMPLE64_AUTOUPDATE})
    add_definitions(-D_AUTOUPDATE)
    message("Enabling auto updater")
else()
    message("Not enabling auto updater")
endif()

find_package(SDL2 REQUIRED)
include_directories(${SDL2_INCLUDE_DIRS})

find_package(Qt6 REQUIRED COMPONENTS Widgets WebSockets)
if (UNIX)
    find_package(Qt6 REQUIRED COMPONENTS DBus)
endif (UNIX)

qt_standard_project_setup()

qt_add_resources(simple64-gui imageresources
    FILES icons/simple64.png
)

add_executable(simple64-gui
    mainwindow.ui
    main.cpp
    mainwindow.cpp
    vidext.cpp
    settingsdialog.cpp
    interface/common.cpp
    plugindialog.cpp
    vkwindow.cpp
    workerthread.cpp
    settingclasses.cpp
    interface/core_commands.cpp
    interface/sdl_key_converter.c
    logviewer.cpp
    keypressfilter.cpp
    netplay/createroom.cpp
    netplay/joinroom.cpp
    netplay/waitroom.cpp
)

if (WIN32)
    target_sources(simple64-gui PRIVATE
        osal/osal_dynamiclib_win32.c
    )
    set(app_icon_resource_windows "icons/icon.rc")
    target_sources(simple64-gui PRIVATE ${app_icon_resource_windows})
endif (WIN32)

if (UNIX)
    target_sources(simple64-gui PRIVATE
        osal/osal_dynamiclib_unix.c
    )
    target_link_libraries(simple64-gui PRIVATE Qt6::DBus)
endif (UNIX)

target_compile_options(simple64-gui PRIVATE -march=x86-64-v3)
target_link_options(simple64-gui PRIVATE -march=x86-64-v3)
target_link_libraries(simple64-gui PRIVATE Qt6::Widgets Qt6::WebSockets)

set_target_properties(simple64-gui PROPERTIES
    WIN32_EXECUTABLE ON
    MACOSX_BUNDLE ON
)