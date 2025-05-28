---
date: 2025-05-24
draft: true
categories: 
  - Qt
tags:
  - Qt
authors:
  - admin
---

# 从QWidget进入QML的世界🌍

<!-- more -->

## CMake

使用传统路径加载QML

```c++
QQmlApplicationEngine engine;
const QUrl url(QStringLiteral("qrc:/qml/main.qml"));
engine.load(url);
```

使用模块加载QML

```c++
engine.loadFromModule("QmlDemo", "main")
```

```cmake
qt_add_qml_module(${PROJECT_NAME}
    URI QmlDemo
    VERSION 1.0
    QML_FILES Main.qml
)

target_link_libraries(${PROJECT_NAME} 
    Qt::Quick
)
```

## QML

### Window介绍


## C++ 与 QML 交互

