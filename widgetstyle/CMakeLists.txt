cmake_minimum_required(VERSION 3.20)
project(lingmostyle)

set(TARGET lingmostyle)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

find_package(ECM REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

set(QT Core Gui Widgets DBus)
find_package(Qt6 REQUIRED ${QT})
find_package(PkgConfig REQUIRED)

find_package(KF6WindowSystem QUIET)
if(NOT KF6WindowSystem_FOUND)
    find_package(KF5WindowSystem REQUIRED)
endif()

include(ECMQueryQt)

set(SRCS
    blurhelper.cpp
    blurhelper.h
    pstyleplugin.cpp
    pstyleplugin.h
    basestyle.h
    basestyle.cpp
    phantomcolor.h
    phantomcolor.cpp
    shadowhelper.h
    shadowhelper.cpp
    tileset.h
    tileset.cpp
    boxshadowrenderer.h
    boxshadowrenderer.cpp
)

add_library(${TARGET} MODULE ${SRCS})

set(_LINK_LIBS
    Qt6::GuiPrivate
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
    Qt6::DBus
)

if(KF6WindowSystem_FOUND)
    list(APPEND _LINK_LIBS KF6::WindowSystem)
elseif(KF5WindowSystem_FOUND)
    list(APPEND _LINK_LIBS KF5::WindowSystem)
endif()

target_link_libraries(${TARGET} ${_LINK_LIBS})

ecm_query_qt(CMAKE_INSTALL_QTPLUGINDIR QT_INSTALL_PLUGINS)

install(TARGETS ${TARGET} DESTINATION ${CMAKE_INSTALL_QTPLUGINDIR}/styles/)
