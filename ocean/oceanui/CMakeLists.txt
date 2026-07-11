################# Qt/KDE #################
find_package(Qt6 REQUIRED CONFIG COMPONENTS Widgets DBus)

# KDE Framework 6 dependencies
find_package(KF6 REQUIRED COMPONENTS
    I18n
    Config
    GuiAddons
    IconThemes
    ConfigWidgets
    WindowSystem)

find_package(Qt6 COMPONENTS Quick)
set(LINGMO_HAVE_QTQUICK ${Qt6Quick_FOUND})

find_package(KF6FrameworkIntegration QUIET CONFIG)
set(LINGMO_HAVE_KSTYLE ${KF6FrameworkIntegration_FOUND})

find_package(XCB COMPONENTS XCB)

find_package(KF6Wayland QUIET CONFIG)
set(LINGMO_HAVE_KWAYLAND ${KF6Wayland_FOUND})

if(UNIX AND NOT APPLE)
    set(LINGMO_HAVE_X11 ${XCB_XCB_FOUND})
else()
    set(LINGMO_HAVE_X11 FALSE)
endif()


################# includes #################
include_directories(
  animations
  debug
)

include_directories(${CMAKE_SOURCE_DIR}/liboceancommon)
include_directories(${CMAKE_BINARY_DIR}/liboceancommon)

################# configuration #################
configure_file(config-ocean.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-ocean.h )

########### next target ###############
set(ocean_PART_SRCS
    animations/oceananimation.cpp
    animations/oceananimations.cpp
    animations/oceananimationdata.cpp
    animations/oceanbaseengine.cpp
    animations/oceanbusyindicatordata.cpp
    animations/oceanbusyindicatorengine.cpp
    animations/oceandialdata.cpp
    animations/oceandialengine.cpp
    animations/oceanenabledata.cpp
    animations/oceangenericdata.cpp
    animations/oceanheaderviewdata.cpp
    animations/oceanheaderviewengine.cpp
    animations/oceanscrollbardata.cpp
    animations/oceanscrollbarengine.cpp
    animations/oceanspinboxengine.cpp
    animations/oceanspinboxdata.cpp
    animations/oceanstackedwidgetdata.cpp
    animations/oceanstackedwidgetengine.cpp
    animations/oceantabbarengine.cpp
    animations/oceantabbardata.cpp
    animations/oceantoolboxengine.cpp
    animations/oceantransitiondata.cpp
    animations/oceantransitionwidget.cpp
    animations/oceanwidgetstateengine.cpp
    animations/oceanwidgetstatedata.cpp
    debug/oceanwidgetexplorer.cpp
    oceanaddeventfilter.cpp
    oceanblurhelper.cpp
    oceanframeshadow.cpp
    oceanhelper.cpp
    oceanmdiwindowshadow.cpp
    oceanmnemonics.cpp
    oceanpropertynames.cpp
    oceanshadowhelper.cpp
    oceansplitterproxy.cpp
    oceanstyle.cpp
    oceanstyleplugin.cpp
    oceantileset.cpp
    oceanwindowmanager.cpp
)

kconfig_add_kcfg_files(ocean_PART_SRCS oceanstyleconfigdata.kcfgc)
add_library(lingmo-kstyle MODULE ${ocean_PART_SRCS})

set(_LINK_LIBS
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
    Qt6::DBus
    KF6::ConfigCore
    KF6::ConfigWidgets
    KF6::GuiAddons
    KF6::IconThemes
    KF6::WindowSystem
)

if(LINGMO_HAVE_QTQUICK)
    list(APPEND _LINK_LIBS Qt6::Quick)
endif()

list(APPEND _LINK_LIBS oceancommon5)

if(LINGMO_HAVE_KSTYLE)
    list(APPEND _LINK_LIBS KF6::Style)
endif()

target_link_libraries(lingmo-kstyle ${_LINK_LIBS})

if(WIN32)
    target_compile_definitions(lingmo-kstyle PRIVATE _USE_MATH_DEFINES _BSD_SOURCE)
endif()

if(LINGMO_HAVE_X11)
    target_link_libraries(lingmo-kstyle ${XCB_LIBRARIES})
endif()

if(LINGMO_HAVE_KWAYLAND)
    target_link_libraries(lingmo-kstyle KF6::WaylandClient)
endif()


########### install files ###############
install(TARGETS lingmo-kstyle DESTINATION ${QT_PLUGIN_INSTALL_DIR}/styles/)
install(FILES ocean.themerc  DESTINATION  ${DATA_INSTALL_DIR}/kstyle/themes)

########### subdirectories ###############
add_subdirectory(config)
