# This repository SORT OF works (very early demo)


## Aim

I would like to be able to use the MapLibre Native Qt widget within PySide6.
For obvious reasons, this is likely the best and prettiest way to render maps.

![really from PySide6](http://stefan.konink.de/maplibre.png)

## State of affairs

There are some crazy things going on with respect to includes, ~~d QtOpenGLWidget~~
The root CMakeLists.txt needs extra include dirs, without them the submodule does not compile.
~~To me it does not make sense that the following line must be added to an unrelated header, but without it Shiboken6 complains.~~
```C
extern Shiboken::Module::TypeInitStruct *SbkPySide6_QtOpenGLWidgetsTypeStructs;
```
~~My CMakeLists.txt in bindings is very taylor made towards "fixing" random compilation failures.~~

~~There are two lines commented out in `bindings2/CMakeLists.txt` which call `utils/pyside_config.py`, because it runs an empty result.~~

~~The only way I could get this to work was to take the cpp file of the upstream repository in my CMakeLists.txt, and effectively compile the .so from there.~~
~~Because I was not able to get the Settings to work (virtually everything of the upstream library must be imported) I changed the header not to have the Settings.~~

```hpp
    explicit GLWidget();
    // explicit GLWidget(const Settings &);
```

~~...and changed the implementation to always come up with the same style, similar to the upstream example.
This is obviously not what we want.~~

```cpp
GLWidget::GLWidget()
    : d_ptr(std::make_unique<GLWidgetPrivate>(this, [] {
        QMapLibre::Styles styles;
        styles.emplace_back("https://demotiles.maplibre.org/style.json", "Demo Tiles");

        QMapLibre::Settings settings;
        settings.setStyles(styles);
        settings.setDefaultZoom(5);
        settings.setDefaultCoordinate(QMapLibre::Coordinate(43, 21));
        return settings;
    }())) {
}
```
(above not required anymore)

So the questions are:
 1. ~~is it possible to just compile the libMapLibreWidgets.so, and link that while specifying which objects Shiboken6 needs to expose.~~ _wrapper.cpp classes need to be added to the compilation.
 2. ~~must the object be rewritten (like a custom wrapper) so we can do the basic stuff like adding and setting styles in Python, or can we prevent this?~~ basic setup works
 3. https://github.com/maplibre/maplibre-native-qt/issues/221 seems to be an upstream bug.

##  How to reproduce the state I am at now on ArchLinux:

Might need #include "types.hpp" in settings.hpp.

```
pacman -S libxml2-legacy

# Fetch the upstream maplibre-native-qt
git submodule update --init --recursive

# Disable the tests, otherwise another include hell which requires full recompilation for every test is at play
sed -i '/add_subdirectory.*test/d' maplibre-native-qt/CMakeLists.txt

# Build the repository without errors
mkdir build
cd build
python -m venv venv
source venv/bin/activate
pip install aqtinstall
aqt install-qt linux desktop 6.9.1
export PATH=`pwd`/6.9.1/gcc_64/bin:$PATH
export CMAKE_MODULE_PATH=`pwd`/6.9.1/gcc_64/lib/cmake
pip install shiboken6-generator pyside6
cmake ..
make
```

## How to do the same on Windows
Install MSVC, make sure that Cmake is also added.

In cmd.exe:
```
"c:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"
aqt install-qt windows desktop 6.9.1 win64_msvc2022_64
pip install pyside6 shiboken6
mkdir build
cmake .. -G "NMake Makefiles" -DCMAKE_PREFIX_PATH="C:\Users\Gebruiker\PycharmProjects\badger\6.9.1\msvc2022_64" -DCMAKE_BUILD_TYPE=Release
nmake
```

Now you would have to collect all the missing dlls.
I have used https://github.com/lucasg/Dependencies to track that down.

