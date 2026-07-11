################# Qt/KDE #################
find_package(Qt6 REQUIRED CONFIG COMPONENTS Widgets DBus)

# Try KF6 first, fall back to KF5
find_package(KF6I18n QUIET)
if(NOT KF6I18n_FOUND)
    find_package(KF5I18n REQUIRED)
endif()

find_package(KF6Config QUIET)
if(NOT KF6Config_FOUND)
    find_package(KF5Config REQUIRED)
endif()

find_package(KF6GuiAddons QUIET)
if(NOT KF6GuiAddons_FOUND)
    find_package(KF5GuiAddons REQUIRED)
endif()

find_package(KF6IconThemes QUIET)
if(NOT KF6IconThemes_FOUND)
    find_package(KF5IconThemes REQUIRED)
endif()

find_package(KF6ConfigWidgets QUIET)
if(NOT KF6ConfigWidgets_FOUND)
    find_package(KF5ConfigWidgets REQUIRED)
endif()

find_package(KF6WindowSystem QUIET)
if(NOT KF6WindowSystem_FOUND)
    find_package(KF5WindowSystem REQUIRED)
endif()

find_package(Qt6 COMPONENTS Quick)
set(OCEAN_HAVE_QTQUICK ${Qt6Quick_FOUND})

find_package(KF6FrameworkIntegration QUIET CONFIG)
if(NOT KF6FrameworkIntegration_FOUND)
    find_package(KF5FrameworkIntegration CONFIG)
endif()
set_package_properties(KF5FrameworkIntegration PROPERTIES
    DESCRIPTION "KF6 Framework Integration"
    URL "https://projects.kde.org/projects/frameworks/frameworkintegration"
    TYPE OPTIONAL
    PURPOSE "Required to use KStyle convenience functionalities in style")

set(OCEAN_HAVE_KSTYLE ${KF5FrameworkIntegration_FOUND})

find_package(XCB COMPONENTS XCB)
set_package_properties(XCB PROPERTIES
    DESCRIPTION "X protocol C-language Binding"
    URL "https://xcb.freedesktop.org"
    TYPE OPTIONAL
    PURPOSE "Required to pass style properties to native Windows on X11 Platform"
)

find_package(KF6Wayland QUIET CONFIG)
if(NOT KF6Wayland_FOUND)
    find_package(KF5Wayland CONFIG)
endif()
set(OCEAN_HAVE_KWAYLAND ${KF5Wayland_FOUND})

if(UNIX AND NOT APPLE)
    set(OCEAN_HAVE_X11 ${XCB_XCB_FOUND})
else()
    set(OCEAN_HAVE_X11 FALSE)
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
add_library(ocean MODULE ${ocean_PART_SRCS})

set(_OCEAN_LINK_LIBS Qt6::Core Qt6::Gui Qt6::Widgets Qt6::DBus)

if(OCEAN_HAVE_QTQUICK)
    list(APPEND _OCEAN_LINK_LIBS Qt6::Quick)
endif()

# KF6 or KF5 libraries
if(KF6Config_FOUND)
    list(APPEND _OCEAN_LINK_LIBS KF6::ConfigCore KF6::ConfigWidgets KF6::GuiAddons KF6::IconThemes KF6::WindowSystem)
else()
    list(APPEND _OCEAN_LINK_LIBS KF5::ConfigCore KF5::ConfigWidgets KF5::GuiAddons KF5::IconThemes KF5::WindowSystem)
endif()

list(APPEND _OCEAN_LINK_LIBS oceancommon5)

if(KF5FrameworkIntegration_FOUND)
    list(APPEND _OCEAN_LINK_LIBS KF6::Style)
endif()

target_link_libraries(ocean ${_OCEAN_LINK_LIBS})

if (WIN32)
    target_compile_definitions(ocean PRIVATE _USE_MATH_DEFINES _BSD_SOURCE)
endif()

if(OCEAN_HAVE_X11)
    target_link_libraries(ocean ${XCB_LIBRARIES})
endif()

if(OCEAN_HAVE_KWAYLAND)
    if(KF6Wayland_FOUND)
        target_link_libraries(ocean KF6::WaylandClient)
    else()
        target_link_libraries(ocean KF5::WaylandClient)
    endif()
endif()


########### install files ###############
install(TARGETS ocean DESTINATION ${QT_PLUGIN_INSTALL_DIR}/styles/)
install(FILES ocean.themerc  DESTINATION  ${DATA_INSTALL_DIR}/kstyle/themes)

########### subdirectories ###############
add_subdirectory(config)
