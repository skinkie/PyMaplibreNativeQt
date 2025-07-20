set(MODULE_NAME pyqmaplibre)
set(PYTHON_MODULE_NAME pyqmaplibre)

set(GENERATED_DIR ${CMAKE_CURRENT_BINARY_DIR}/generated)
file(MAKE_DIRECTORY ${GENERATED_DIR})

set(shiboken_generator_path shiboken6)

add_custom_command(
    OUTPUT
        ${GENERATED_DIR}/${MODULE_NAME}/${PYTHON_MODULE_NAME}_module_wrapper.cpp
    COMMAND ${shiboken_generator_path}
        --generator-set=shiboken
        --enable-pyside-extensions
        --enable-parent-ctor-heuristic
        --use-isnull-as-nb_nonzero
        --avoid-protected-hack
        --include-paths=${CMAKE_SOURCE_DIR}/maplibre-native-qt/include
        --include-paths=${CMAKE_SOURCE_DIR}/maplibre-native-qt/src/widgets
        --include-paths=${CMAKE_SOURCE_DIR}/maplibre-native-qt/src/core
        --include-paths=${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets/include
        --include-paths=${CMAKE_BINARY_DIR}/maplibre-native-qt/src/core/include
	--include-paths=${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets/include/QMapLibreWidgets
	--include-paths=${CMAKE_SOURCE_DIR}/bindings
        --include-paths=/usr/include/qt6
        --include-paths=/usr/include/qt6/QtWidgets
        --include-paths=/usr/include/qt6/QtOpenGLWidgets
        --include-paths=/usr/lib/gcc/x86_64-pc-linux-gnu/15/include
        --typesystem-paths=${CMAKE_CURRENT_SOURCE_DIR}
        --typesystem-paths=/usr/share/PySide6/typesystems
        --output-directory=${GENERATED_DIR}
        ${CMAKE_SOURCE_DIR}/bindings/${PYTHON_MODULE_NAME}_global.h
        ${CMAKE_CURRENT_SOURCE_DIR}/${PYTHON_MODULE_NAME}_module.xml
    DEPENDS
        ${CMAKE_SOURCE_DIR}/bindings/${PYTHON_MODULE_NAME}_global.h
        ${CMAKE_CURRENT_SOURCE_DIR}/${PYTHON_MODULE_NAME}_module.xml
    COMMENT "Running Shiboken6 to generate Python bindings"
)

add_library(${MODULE_NAME} MODULE
    ${GENERATED_DIR}/${PYTHON_MODULE_NAME}/${PYTHON_MODULE_NAME}_module_wrapper.cpp
)

target_include_directories(${MODULE_NAME}
    PRIVATE
        ${CMAKE_SOURCE_DIR}/maplibre-native-qt/include
	/usr/include/shiboken6
	/usr/include/python3.13
	/usr/include/PySide6
	/usr/include/PySide6/QtCore
	/usr/include/PySide6/QtGui
	/usr/include/PySide6/QtWidgets
	/usr/include/qt6/QtOpenGLWidgets
	${CMAKE_SOURCE_DIR}/maplibre-native-qt/src/core
	${CMAKE_BINARY_DIR}/maplibre-native-qt/src/core/include
        ${CMAKE_SOURCE_DIR}/maplibre-native-qt/include
	${CMAKE_SOURCE_DIR}/maplibre-native-qt/src/widgets
	${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets
        ${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets/include
	${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets/include/QMapLibreWidgets
	${CMAKE_SOURCE_DIR}/bindings
	${GENERATED_DIR}
)

find_package(PySide6 REQUIRED COMPONENTS Core Widgets OpenGLWidgets)

target_link_libraries(${MODULE_NAME}
    PRIVATE
        Qt6::Core
        Qt6::Gui
	Qt6::Widgets
	Qt6::OpenGLWidgets

	${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets/libQMapLibreWidgets.a
	${CMAKE_BINARY_DIR}/maplibre-native-qt/src/core/libQMapLibre.a
	${CMAKE_BINARY_DIR}/maplibre-native-qt/src/location/plugins/libqtgeoservices_maplibre.a
	${CMAKE_BINARY_DIR}/maplibre-native-qt/src/location/libQMapLibreLocation.a
	# ${CMAKE_BINARY_DIR}/maplibre-native-qt/src/widgets/libQMapLibreWidgets.so
)

set_target_properties(${MODULE_NAME} PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}
    PREFIX ""  # geen 'lib' prefix
    OUTPUT_NAME ${PYTHON_MODULE_NAME}  # QMapLibre.so
)

# === Voor Python-zoekpad testen ===
add_custom_target(copy_bindings ALL
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:${MODULE_NAME}> ${CMAKE_BINARY_DIR}/${PYTHON_MODULE_NAME}.so
    DEPENDS ${MODULE_NAME}
)
