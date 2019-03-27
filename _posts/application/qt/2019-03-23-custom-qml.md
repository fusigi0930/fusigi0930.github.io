---
layout: post
title: Create custom QML for QT
description: custom QML
tags: QT javascript
---

# Create a QML can be import by other QML
## description
to avoid a QML file too height, and create re-usable module, I introduction 2 methods in here:
* QML built-in
* QML in rcc file

## notice
there are some points and both of built-in and rcc file are almost the same:
* qmldir file
* the root of QML module, need use Item QML object, it will let QML editor parser correct

## How to do
### Built-in
* create a qml file (e.g. extend.qml) with content:

```javascript
import QtQuick 2.0
import QtQuick.Controls 2.0

Item {
  Rectangle {
    anchors.fill: parent
    ...
  }
}
```

* create a qmldir file with content:

```
ExtendObj 1.0 extend.qml
```

* add qml file and qmldir file into QRC file with prefix (e.g. modules)

```xml
<RCC>
    <qresource prefix="/">
        <file>main.qml</file>
    </qresource>
    <qresource prefix="/modules">
        <file>extend.qml</file>
        <file>qmldir</file>
    </qresource>
</RCC>
```

* import the ExtendObj in main.qml (or other qml file):

```javascript
import QtQuick 2.0
import QtQuick.Window 2.2
import "qrc:/modules"

Window {
    visible: true
    width: 640
    height: 480
    title: qsTr("Hello World")

    ExtendObj {
        anchors.fill: parent
        ...
    }
}

```

now, you can use the QML module that is built by yourself.

### external RCC filo

we only adjust some procedure from built-in methods (still use extend.qml and qmldir)

* create a new qrc file (e.g. common.qrc)

```xml
<RCC>
    <qresource prefix="/modules">
        <file>extend.qml</file>
        <file>qmldir</file>
    </qresource>
</RCC>
```

* build rcc file by using QT utility "rcc"

```shell
rcc -binary common.qrc -o common.rcc
```

* load rcc while start program (load it in main.cpp), the point is runtime loading and working directory

```c++
#include <QResource>
...
...
QResource::registerResource("common.rcc")
```

and you can use the same method to call your modules in main.qml (or other QML files).
