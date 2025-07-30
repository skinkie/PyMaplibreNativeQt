cmake_minimum_required(VERSION 3.18)
project(QMapLibreBindings LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(MLN_QT_WITH_LOCATION "Enable location support" OFF)
option(MLN_QT_WITH_WIDGETS "Enable widgets" ON)
option(MLN_QT_STATIC "Build as static library" OFF)
option(MLN_WITH_RTTI "Build with RTTI" ON)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/maplibre-native-qt/src/core/style ${CMAKE_CURRENT_SOURCE_DIR}/maplibre-native-qt/test/common/ ${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets)

add_subdirectory(maplibre-native-qt)

find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets OpenGLWidgets Sql Network)
find_package(Python3 REQUIRED COMPONENTS Interpreter Development)
# find_package(ICU REQUIRED COMPONENTS uc)

add_subdirectory(bindings2)
