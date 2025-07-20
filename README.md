# This repository DOES NOT work


## Aim

I would like to be able to use the MapLibre Native Qt widget within PySide6.
For obvious reasons, this is likely the best and prettiest way to render maps.


## State of affairs

There are some crazy things going on with respect to includes, and QtOpenGLWidget.
The root CMakeLists.txt needs extra include dirs, without them the submodule does not compile.
To me it does not make sense that the following line must be added to an unrelated header, but without it Shiboken6 complains.
```C
extern Shiboken::Module::TypeInitStruct *SbkPySide6_QtOpenGLWidgetsTypeStructs;
```
My CMakeLists.txt in bindings is very taylor made towards "fixing" random compilation failures.


How to reproduce the state I am at now on ArchLinux:


```
# Fetch the upstream maplibre-native-qt
git submodule update --init --recursive

# Disable the tests, otherwise another include hell which requires full recompilation for every test is at play
sed -i '/add_subdirectory.*test/d' maplibre-native-qt/CMakeLists.txt

# Build the repository without errors
mkdir build
cd build
cmake ..
make
```

