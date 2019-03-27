# Deploy QT application for windows
## description
QT has a utility can help you to prepare all of dependence dynamic libraries, it can let you
deploy application more convenient.

## pre-condition
1. assume QT source code in path &lt;src_path&gt;, and all of QML files are stored in &lt;src_path&gt;\qml
2. a built application file &lt;application&gt; and stored in &lt;release&gt; folder
3. QT utilities are stored in &lt;qt_common_folder&gt;/bin (e.g. c:\Qt\x.xx\mingw_xx\bin)

## procedure
```bat
cd <release>
<qt_common_folder>\bin\windeployqt.exe --qml_dir <src_path>\qml <application>
```

this utility will copy the common libraries to the release folder
