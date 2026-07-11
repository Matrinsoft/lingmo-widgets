cmake_minimum_required(VERSION 3.20)
project(lingmo-kstyle VERSION 0.43.0 DESCRIPTION "Lingmo KStyle" LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(GenerateExportHeader)
include(WriteBasicConfigVersionFile)
include(FeatureSummary)

# Try KF6 first, fall back to KF5
find_package(ECM REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR} ${CMAKE_SOURCE_DIR}/cmake)

include(ECMInstallIcons)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(KDEClangFormat)

include(GtkUpdateIconCache)

add_subdirectory(colors)
add_subdirectory(liboceancommon)
add_subdirectory(oceanui)
add_subdirectory(misc)

include(ECMSetupVersion)
ecm_setup_version(${PROJECT_VERSION} VARIABLE_PREFIX OCEAN
                  PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/OceanConfigVersion.cmake"
                  )

# create a Config.cmake and a ConfigVersion.cmake file and install them
set(CMAKECONFIG_INSTALL_DIR "${CMAKECONFIG_INSTALL_PREFIX}/Ocean")

ecm_configure_package_config_file("${CMAKE_CURRENT_SOURCE_DIR}/OceanConfig.cmake.in"
                                  "${CMAKE_CURRENT_BINARY_DIR}/OceanConfig.cmake"
                                  PATH_VARS KDE_INSTALL_FULL_DATADIR
                                  INSTALL_DESTINATION  ${CMAKECONFIG_INSTALL_DIR}
                                  )

# add clang-format target for all our real source files
file(GLOB_RECURSE ALL_CLANG_FORMAT_SOURCE_FILES *.cpp *.h)
kde_clang_format(${ALL_CLANG_FORMAT_SOURCE_FILES})

install(FILES  "${CMAKE_CURRENT_BINARY_DIR}/OceanConfig.cmake"
               "${CMAKE_CURRENT_BINARY_DIR}/OceanConfigVersion.cmake"
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel
        )

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
