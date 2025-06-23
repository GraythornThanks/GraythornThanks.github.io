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

# QML与C++的集成

<!-- more -->

## 概述

将 C++ 对象/值暴露给 QML (多数场景)

- 可实例化对象/组件：qmlRegisterType + Q_PROPERTY + Q_INVOKABLE + signals (首选)
- 全局单例服务： qmlRegisterSingletonType 或 qmlRegisterSingletonInstance
- 简单全局对象/常量：QQmlContext::setContextProperty
- 自动暴露：Q_ENUM

在 C++ 中访问和操作 QML 对象 （应该尽量避免）

- 操作已知稳定UI：findChild + QMetaObject::invokeMethod / setProperty
- 动态创建复杂UI：QQmlComponent
- QQmlExtension
- QQmlContext::get()

## C++ 暴露给 QML

### 把 C++ 类的属性公开给 QML

QML 引擎能通过元对象系统内省QObject 实例。这意味着任何 QML 代码都能访问QObject 派生类实例的以下成员：

- 属性：Q_PROPERTY
- 方法：公共槽函数或标记为 Q_INVOKABLE
- 信号：signals

> 使用`Q_ENUM`声明的枚举也是可见的

#### 公开属性

```cpp
class Message : public QObject
{
    Q_OBJECT
    QML_ELEMENT
    Q_PROPERTY(QString author READ author WRITE setAuthor NOTIFY authorChanged)
    // ...
};
```

要点：

在 C++ 类声明中添加 QML_ELEMENT 宏后，构建系统会自动生成必要的元类型注册代码，而无需手动调用qmlRegisterType()
任何可写属性都应该有一个相关的NOTIFY信号

##### 当嵌套对象类型属性时，需要子类型也在类型系统中注册

```qml
Message {
  body: MessageBody { // MessageBody类型也需要在类型系统中注册
    text: "Hello, World!"
  }
}
```

##### 具有对象列表类型的属性

包含 QObject 派生类型列表的属性也可以暴露给 QML, 但是 QList 不是 QObject-derived 类型

```cpp
Q_PROPERTY(QQmlListProperty<Message> messages READ messages)
```

##### 分组属性

任何只读对象类型属性都可作为分组属性从 QML 代码中访问。这可用于公开一组相关属性，描述一个类型的一组属性

```cpp
class Message : public QObject
{
    Q_OBJECT
    Q_PROPERTY(MessageAuthor* author READ author)
    // ...
}
```

可以使用 QML 中的分组属性语法写入author 属性

```qml
Message {
    author.name: "Alexandra"
    author.email: "alexandra@mail.com"
}
```

#### 公开方法

能够被 QML 调用的 C++ 方法：

- 使用 Q_INVOKABLE 宏标记的公共方法
- 公共槽函数

```cpp
class MessageBoard : public QObject
{
    Q_OBJECT
    QML_ELEMENT

public:
    Q_INVOKABLE bool postMessage(const QString &msg) {
        qDebug() << "Called the C++ method with" << msg;
        return true;
    }

public slots:
    void refresh() {
        qDebug() << "Called the C++ slot";
    }
};
```

#### 公开信号

QML 可以访问 QObject 派生类的任何公共信号

QML 引擎会自动为<signal>创建一个信号处理函数<onSignal>

```c++
class MessageBoard : public QObject
{
    Q_OBJECT
public:
   // ...
signals:
   void newMessagePosted(const QString &subject);
};
```

```qml
MessageBoard {
    onNewMessagePosted: console.log("New message posted: " + subject)
}
```

### 从 C++ 向 QML 公开状态

#### 使用单例

将一些全局属性公开给 QML
将QML_ELEMENT或QML_NAMEED_ELEMENT宏和QML_SINGLETON宏添加到要公开的类中

```cpp
// Singleton.h
class Singleton : public QObject
{
    Q_OBJECT
    Q_PROPERTY(int thing READ thing WRITE setThing NOTIFY thingChanged FINAL)
    QML_ELEMENT
    QML_SINGLETON

public:
    Singleton(QObject *parent = nullptr) : QObject(parent) {}

    int thing() const { return m_value; }
    void setThing(int v)
    {
        if (v != m_value) {
            m_value = v;
            emit thingChanged();
        }
    }

signals:
    void thingChanged();

private:
    int m_value = 12;
};
```

现在，可以在 QML 中访问该单例的属性

```qml
QtObject {
    objectName: "The thing is " + Singleton.thing
}
```

### 从 C++ 定义 QML 类型

所有 QML 对象都是 QObject 派生类型

#### 从 C++ 加载 QML 对象

QML 可通过 QQmlComponent 或 QQuickView 加载为 C++ 对象

```C++
// Using QQmlComponent
QQmlEngine engine;
QQmlComponent component(&engine, QUrl::fromLocalFile("path/to/your.qml"));
QObject *object = component.create();
```

```C++
// Using QQuickView
QQuickView view;
view.setSource(QUrl::fromLocalFile("path/to/your.qml"));
view.show();
```

现在，可以使用 QObject::setProperty() 或 QQmlProperty::write() 来修改属性

#### 通过定义明确的 C++ 接口访问 QML 对象

定义一个C++接口

TODO

## 在 C++ 和 QML 之间选择正确的集成方法

```mermaid
graph TD

```

## QML 和 C++ 之间的数据交换

[参考 doc.qt.io](https://doc.qt.io/qt-6/zh/qtqml-cppintegration-data.html)

### 数据所有权

从 C++ 传输到 QML 时，数据的所有权始终属于 C++

例外：这个规则的例外是当一个QObject 从一个明确的 C++ 方法调用返回时：在这种情况下，QML 引擎承担对象的所有权，除非对象的所有权已明确设置为保留在 C++ 中，通过调用QQmlEngine::setObjectOwnership() 并指定了 QQmlEngine::CppOwnership

### 基本 Qt 数据类型

| Qt类型 | QML值类型 |
| --- | --- |
| bool | bool |
| uint,int | int |
| double | double |
| float | real |
| QString | string |
| QUrl | url |
| QColor | color |
| QSize,QSizeF | size |
| QRect,QRectF | rect |
| QPoint,QPointF | point |
| QDateTime | date |

### QObject派生类型



## 