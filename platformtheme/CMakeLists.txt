cmake_minimum_required(VERSION 3.20)
project(lingmo-platformtheme VERSION 1.0.0 DESCRIPTION "Lingmo Platform Theme" LANGUAGES CXX)
include(GNUInstallDirs)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

find_package(Qt6Core REQUIRED)
find_package(Qt6Widgets REQUIRED)
find_package(Qt6DBus REQUIRED)
find_package(Qt6 REQUIRED)
find_package(Qt6Gui CONFIG REQUIRED)

# Optional Qt6 modules
find_package(Qt6QuickControls2 QUIET)

# KDE Framework 6 dependencies
find_package(KF6WindowSystem QUIET)
if(NOT KF6WindowSystem_FOUND)
    message(STATUS "KF6WindowSystem not found, skipping lingmo-platformtheme")
    return()
endif()

find_package(PkgConfig REQUIRED)
pkg_check_modules(XCB_EWMH REQUIRED xcb xcb-ewmh x11)

set(SRCS
    main.cpp
    platformtheme.h
    platformtheme.cpp
    hintsettings.h
    hintsettings.cpp
    systemtrayicon.h
    systemtrayicon.cpp
    qdbusmenubar_p.h
    qdbusmenubar.cpp
    x11integration.h
    x11integration.cpp
    statusnotifier/dbustypes.h
    statusnotifier/dbustypes.cpp
    statusnotifier/statusnotifieritem.h
    statusnotifier/statusnotifieritem.cpp
)

qt_add_dbus_interface(SRCS org.kde.StatusNotifierWatcher.xml statusnotifierwatcher_interface)

qt_add_dbus_adaptor(SRCS
    statusnotifier/org.kde.StatusNotifierItem.xml
    statusnotifier/statusnotifieritem.h
    StatusNotifierItem
)

add_library(lingmoplatformtheme MODULE ${SRCS})

target_compile_definitions(lingmoplatformtheme
    PRIVATE
        "QT_NO_FOREACH"
        "LIB_FM_QT_SONAME=\"${LIB_FM_QT_SONAME}\""
)

set(_LINK_LIBS
    Qt6::GuiPrivate
    Qt6::Widgets
    Qt6::Core
    Qt6::DBus
    KF6::WindowSystem
    ${XCB_EWMH_LIBRARIES}
)

if(Qt6QuickControls2_FOUND)
    list(APPEND _LINK_LIBS Qt6::QuickControls2)
endif()

target_link_libraries(lingmoplatformtheme PRIVATE ${_LINK_LIBS})

target_include_directories(lingmoplatformtheme PRIVATE
    ${XCB_EWMH_INCLUDE_DIRS}
)

get_target_property(QT_QMAKE_EXECUTABLE ${Qt6Core_QMAKE_EXECUTABLE} IMPORTED_LOCATION)
if(NOT QT_QMAKE_EXECUTABLE)
    message(FATAL_ERROR "qmake is not found.")
endif()

execute_process(COMMAND ${QT_QMAKE_EXECUTABLE} -query QT_INSTALL_PLUGINS
    OUTPUT_VARIABLE QT_PLUGINS_DIR
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
if(QT_PLUGINS_DIR)
    message(STATUS "Qt6 plugin directory: ${QT_PLUGINS_DIR}")
else()
    message(FATAL_ERROR "Qt6 plugin directory cannot be detected.")
endif()

install(TARGETS lingmoplatformtheme LIBRARY DESTINATION "${QT_PLUGINS_DIR}/platformthemes")
