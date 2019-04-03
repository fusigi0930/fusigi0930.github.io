---
layout: post
title: Interact between QML and C++ layer
description: introduction for how to communicate between c++ layer and qml layer
tags: QT javascript c++
---

# QML and C++
while creating a Qt quick application, it is impossible to avoid communicate between qml and c++, it can be shown like:
<p class="full-width"><img src="/public/image/qt-qml-app.png" /></p>
there are some methods, and I like the first one:
## call c++ code from qml
### create object
* a class is inherited from QObject
you need create a c++ object and inherited from QObject:

```cpp
class CInterface : public QObject {
  ...
}
```

* register class in main.cpp
after you created the class, you need register it in main.cpp

```cpp
#include <QQmlEngine>
...
qmlRegisterType<CInterface>("interface", 1, 0, "interface");
```

* use in qml
finally, you just add the code in your qml file like:
```javascript
import interface 1.0
...
interface {
  ...
}
```

### communicate between them
* QT signal and slot
>* c++ signal to qml
we assume our object has a signal sigAction like this:
```cpp
class CInterface : public QObject {
  ...
signals:
  void sigAction();
}
```

then we use it in qml:
```javascript
interface {
  id: c_interface
  onSigAction: {
    ...
  }
}
```
>* qml signal to c++
we assume our object has a slot slotAction in C++ and a signal sigQml in QML:
```cpp
class CInterface : public QObject {
  ...
public slots:
  void slotAction();
}
```

and the QML part:
```javascript
interface {
  id: c_interface
}
...
Component.onCompleted: {
  c_interface.slotAction();
}
```

* Q_INVOKABLE declaration
the Q_INVOKABLE declaration can let QML call c++ method directly:
(c++ part)
```c++
class CInterface : public QObject {
  ...
public:
  Q_INVOKABLE void action();
}
```

and the QML part:
```javascript
interface {
  id: c_interface
}
...
Component.onCompleted: {
  c_interface.action();
}
```

### type between qml and c++
there are usually used types between c++ and qml, you can refer [here](http://doc.qt.io/qt-5/qtqml-cppintegration-data.html) to find you want to use:

Qt Type | QML Type
--------|---------
bool | bool
unsigned int, int | int
double | double
float, qreal | real
QString | string
QVariant | var


## call qml from c++
in this case, you have to set the objectName for the object you want to call from c++
```javascript
Rectangle {
  id: "rect"
  objectName: "rectObj"
  ...
}
```
then you can find the object from c++
```cpp
QQuickItem *item = engine.findChild<QQuickItem*>("rectObj");
```
