################# Qt/KDE #################
find_package(Qt6 REQUIRED CONFIG COMPONENTS Widgets DBus)

# All KDE Framework dependencies - optional, build what we can
find_package(KF6I18n QUIET)
find_package(KF5I18n QUIET)
find_package(KF6Config QUIET)
find_package(KF5Config QUIET)
find_package(KF6GuiAddons QUIET)
find_package(KF5GuiAddons QUIET)
find_package(KF6IconThemes QUIET)
find_package(KF5IconThemes QUIET)
find_package(KF6ConfigWidgets QUIET)
find_package(KF5ConfigWidgets QUIET)
find_package(KF6WindowSystem QUIET)
find_package(KF5WindowSystem QUIET)

find_package(Qt6 COMPONENTS Quick)
set(LINGMO_HAVE_QTQUICK ${Qt6Quick_FOUND})

find_package(KF6FrameworkIntegration QUIET CONFIG)
find_package(KF5FrameworkIntegration QUIET CONFIG)

set(LINGMO_HAVE_KSTYLE ${KF5FrameworkIntegration_FOUND})

find_package(XCB COMPONENTS XCB)

find_package(KF6Wayland QUIET CONFIG)
find_package(KF5Wayland QUIET CONFIG)
set(LINGMO_HAVE_KWAYLAND ${KF5Wayland_FOUND})

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

set(_LINK_LIBS Qt6::Core Qt6::Gui Qt6::Widgets Qt6::DBus)

if(LINGMO_HAVE_QTQUICK)
    list(APPEND _LINK_LIBS Qt6::Quick)
endif()

# KF6 or KF5 libraries
if(KF6Config_FOUND)
    list(APPEND _LINK_LIBS KF6::ConfigCore KF6::ConfigWidgets KF6::GuiAddons KF6::IconThemes KF6::WindowSystem)
elseif(KF5Config_FOUND)
    list(APPEND _LINK_LIBS KF5::ConfigCore KF5::ConfigWidgets KF5::GuiAddons KF5::IconThemes KF5::WindowSystem)
endif()

list(APPEND _LINK_LIBS oceancommon5)

if(LINGMO_HAVE_KSTYLE)
    if(KF6FrameworkIntegration_FOUND)
        list(APPEND _LINK_LIBS KF6::Style)
    elseif(KF5FrameworkIntegration_FOUND)
        list(APPEND _LINK_LIBS KF5::Style)
    endif()
endif()

target_link_libraries(lingmo-kstyle ${_LINK_LIBS})

if(WIN32)
    target_compile_definitions(lingmo-kstyle PRIVATE _USE_MATH_DEFINES _BSD_SOURCE)
endif()

if(LINGMO_HAVE_X11)
    target_link_libraries(lingmo-kstyle ${XCB_LIBRARIES})
endif()

if(LINGMO_HAVE_KWAYLAND)
    if(KF6Wayland_FOUND)
        target_link_libraries(lingmo-kstyle KF6::WaylandClient)
    elseif(KF5Wayland_FOUND)
        target_link_libraries(lingmo-kstyle KF5::WaylandClient)
    endif()
endif()


########### install files ###############
install(TARGETS lingmo-kstyle DESTINATION ${QT_PLUGIN_INSTALL_DIR}/styles/)
install(FILES ocean.themerc  DESTINATION  ${DATA_INSTALL_DIR}/kstyle/themes)

########### subdirectories ###############
add_subdirectory(config)
