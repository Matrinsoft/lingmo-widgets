cmake_minimum_required(VERSION 3.20)

project(lingmo-widgets VERSION 1.0.0 DESCRIPTION "Lingmo Widgets" LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

add_subdirectory(platformtheme)
add_subdirectory(widgetstyle)
add_subdirectory(ocean)