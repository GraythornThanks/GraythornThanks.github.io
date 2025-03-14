---
date: 2024-12-31
draft: true
categories:
  - OpenGL
  - C++
tags:
  - OpenGL
  - C++
authors:
  - admin
---

# Windows 配置 OpenGL 环境

经验包

<!-- more -->

## 参考资料

网站

- [LearnOpenGL](https://learnopengl-cn.github.io/)
- [OpenGL](https://www.opengl.org/)
- [OpenGL Registry](https://registry.khronos.org/OpenGL/index_gl.php)
- [opengl接口版本支持](https://docs.gl/)

书籍

- 红宝书 《OpenGL 编程指南》

## 使用 GLFW + GLAD

### 1. 使用CMake从源码构建

下载：

[GLFW](https://www.glfw.org/)
[GLAD](https://glad.dav1d.de/)

添加到CMake:

```CMake
add_subdirectory(GLFW)
add_subdirectory(GLAD)
```

链接库：

```CMake
target_link_libraries(main PRIVATE glfw glad)
```

将生成的库和可执行文件放置到同一个文件夹下

```CMake
set(DEBUG_OUTPUT_DIR ${PROJECT_BINARY_DIR}/debug/bin)
set(RELEASE_OUTPUT_DIR ${PROJECT_BINARY_DIR}/release/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${DEBUG_OUTPUT_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${DEBUG_OUTPUT_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${DEBUG_OUTPUT_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${RELEASE_OUTPUT_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${RELEASE_OUTPUT_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${RELEASE_OUTPUT_DIR})
```

!!! note "注意"
    构建glfw和glad时主项目应该为C/C++项目 `project(OpenGLWidget LANGUAGES C CXX)`

### 2. 使用CMake从包管理器构建

使用vcpkg安装

```pwsh
$ vcpkg install glfw3:x64-windows
$ vcpkg install glad:x64-windows
```

集成到CMake：

```CMake
find_package(glfw3 REQUIRED)
find_package(glad REQUIRED)
```

链接库：

```CMake
target_link_libraries(main PRIVATE glfw glad)
```

## 使用 Qt

### 使用 `QOpenGLWidget` 替换GLFW

!!! note 
    QOpenGLWidget 提供了三个便捷的虚函数，可以重载，用来重新实现典型的OpenGL任务

    - paintGL：渲染OpenGL场景。widget 需要更新时调用
    - resizeGL：设置OpenGL视口、投影等。widget 调整大小（或首次显示）时调用
    - initializeGL：设置OpenGL资源和状态。第一次调用 resizeGL() / paintGL() 之前调用一次

> 如果需要从`paintGL()`以外的位置触发重新绘制（典型示例是使用计时器设置场景动画），则应调用widget的`update()`函数来安排更新。

> 调用`paintGL()`、`resizeGL()`或`initializeGL()`时，widget 的OpenGL呈现上下文将变为当前。如果需要从其他位置（例如，在 widget 的构造函数或自己的绘制函数中）调用标准OpenGL API函数，则必须首先调用`makeCurrent()`然后调用`doneCurrent()`

### 使用 `QOpenGLFunctions_X_Y_Core` 替换GLAD

!!! note
    `QOpenGLFunctions_X_Y_Core` 提供OpenGL X.Y版本核心模式的所有功能，是对OpenGL函数的封装
    `initializeOpenGLFunctions` 初始化OpenGL函数

### CMake 配置

添加库路径

```CMake
if(WIN32)
    if(MSVC)
        set(CMAKE_PREFIX_PATH C:/Qt/6.2.4/msvc2019_64)
        set(Qt6_DIR C:/Qt/6.2.4/msvc2019_64/lib/cmake/Qt6)
    endif()
endif()
```

添加Qt库和所需的组件

```CMake
find_package(Qt6 REQUIRED COMPONENTS Widgets Gui OpenGL OpenGLWidgets)
```

链接库

```CMake
target_link_libraries(QLearn PRIVATE
    ws2_32
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
    Qt6::OpenGL
    Qt6::OpenGLWidgets)
```

使用 windeployqt 部署 Qt 保证能找到 Qt 的库

```CMake
function(deploy_qt_runtime TARGET_NAME)
    if(WIN32)
        get_target_property(_qmake_executable Qt${QT_VERSION_MAJOR}::qmake IMPORTED_LOCATION)
        get_filename_component(_qt_bin_dir "${_qmake_executable}" DIRECTORY)
        find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS "${_qt_bin_dir}")

        if(NOT WINDEPLOYQT_EXECUTABLE)
            message(FATAL_ERROR "windeployqt not found!")
        endif()

        add_custom_command(TARGET ${TARGET_NAME} POST_BUILD
            COMMAND "${WINDEPLOYQT_EXECUTABLE}"
                --no-compiler-runtime
                --no-translations
                --no-system-d3d-compiler
                $<TARGET_FILE:${TARGET_NAME}>
            COMMENT "Deploying Qt dependencies for ${TARGET_NAME}..."
        )
    endif()
endfunction()

deploy_qt_runtime(QLearn)
```




