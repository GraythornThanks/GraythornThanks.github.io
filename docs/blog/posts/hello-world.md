---
date: 2024-12-23
draft: true
categories:
  - other
tags:
  - Hello World
  - Markdown
  - 示例
authors:
  - admin
---

# Hello World！

本篇作为部署时期创建的TestBlog，用于测试MkDocs的能力，将展示一些常用的Markdown语法和MkDocs的功能

<!-- more -->

## Markdown 语法测试
<a id="markdown-syntax-test"></a>

### 1. 文本格式化

Markdown支持多种文本格式化方式：

- **粗体文本** 使用 `**文本**`
- *斜体文本* 使用 `*文本*`
- ~~删除线~~ 使用 `~~文本~~`
- ==高亮== 使用 `==文本==`
- `代码` 使用 `` `代码` ``
- 上标H^2^O 使用 `H^2^O`
- 下标H~2~O 使用 `H~2~O`
- ++ctrl+c++ 使用 `++ctrl+c++`

### 2. 列表

#### 无序列表
- 项目1
- 项目2
  - 子项目2.1
  - 子项目2.2
- 项目3

#### 有序列表
1. 第一步
2. 第二步
3. 第三步

### 3. 引用

> 这是一个引用示例
> 
> 可以包含多行文本
> 
> — *引用作者*

### 4. 代码块
  
下面是一个Python代码示例：

```python
def hello_world():
    print("Hello, World!")
    
if __name__ == "__main__":
    hello_world()
```

### 5. 表格

| 功能     | 语法           | 效果     |
|----------|---------------|----------|
| 粗体     | `**文本**`    | **文本** |
| 斜体     | `*文本*`      | *文本*   |
| 代码     | `` `代码` ``  | `代码`   |

### 6. 提示框

!!! note "提示"
    这是一个提示框，用于显示重要信息。

!!! warning "警告"
    这是一个警告框，用于显示警告信息。

!!! tip "技巧"
    这是一个技巧框，用于分享有用的技巧。

!!! danger "危险"
    这是一个危险框，用于显示危险信息。

!!! abstract "摘要"
    这是一个摘要框，用于显示摘要信息。

!!! info "信息"
    这是一个信息框，用于显示信息。

!!! success "成功"
    这是一个成功框，用于显示成功信息。

!!! failure "失败"
    这是一个失败框，用于显示失败信息。

!!! question "问题"
    这是一个问题框，用于显示问题信息。

!!! example "示例"
    这是一个示例框，用于显示示例信息。

!!! quote "引用"
    这是一个引用框，用于显示引用信息。

### 7. 任务列表

- [x] 创建第一篇博客
- [x] 添加基本格式
- [ ] 添加更多内容
- [ ] 持续更新

### 8. 链接和图片

#### 链接
- [访问GitHub](https://github.com/)
- [查看更多文章](../index.md)

#### 图片
![示例图片](https://source.unsplash.com/random/800x400?blog)

### 9. 数学公式

\[
E = mc^2
\]

行内公式示例：\(f(x) = x^2\)

更多数学公式示例：

\[
\begin{aligned}
\frac{d}{dx}e^x &= e^x \\
\int x^2 dx &= \frac{x^3}{3} + C \\
\sum_{i=1}^n i &= \frac{n(n+1)}{2}
\end{aligned}
\]

### 10. 脚注

一个带有脚注的文本[^1]。

[^1]: 这是脚注的内容。

### 11. 分割线

水平分割线

----------


### 12. 页内跳转

[跳转到](#markdown-syntax-test)


----------

# MkDocs 使用指南

## 1. MkDocs简介

## 2. 安装和基本设置

### 2.1 安装MkDocs

首先，确保已安装Python，然后使用pip安装MkDocs：

```bash
# 创建虚拟环境
python -m venv venv
# 激活虚拟环境
# Windows
.\venv\Scripts\activate
# Linux/Mac
source venv/bin/activate

# 安装MkDocs和Material主题
pip install mkdocs mkdocs-material
```

### 2.2 创建新项目

```bash
# 创建新项目
mkdocs new my-project
cd my-project

# 启动开发服务器
mkdocs serve
```

## 3. 配置文件详解

MkDocs使用`mkdocs.yml`作为配置文件：

### 3.1 基本配置

```yaml
# 网站基本信息
site_name: My Blog                    # 网站名称
site_author: Your Name                # 作者名称
site_description: A personal blog     # 网站描述
```

### 3.2 主题配置

```yaml
theme:
  name: material                      # 使用Material主题
  features:
    - navigation.tabs                 # 顶部导航标签
    - navigation.sections             # 导航分区
    - navigation.top                  # 返回顶部按钮
    - search.suggest                 # 搜索建议
    - search.highlight               # 搜索结果高亮
```

### 3.3 外观定制

```yaml
theme:
  palette:
    # 浅色主题
    - scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    
    # 深色主题
    - scheme: slate
      primary: indigo
      accent: indigo
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
```

## 4. 常用插件

MkDocs支持多种插件来扩展功能：

### 4.1 搜索插件

```yaml
plugins:
  - search                           # 搜索功能
```

### 4.2 博客插件

```yaml
plugins:
  - blog:
      blog_dir: blog
      post_url_format: "{date}/{slug}"
      authors: true
      pagination: true
      pagination_per_page: 5
```

## 5. Markdown扩展

MkDocs支持多种Markdown扩展，增强文档功能：

```yaml
markdown_extensions:
  - pymdownx.highlight            # 代码高亮
  - pymdownx.superfences         # 支持嵌套代码块
  - admonition                   # 提示块
  - footnotes                    # 脚注
```

## 6. 部署

### 6.1 Github Actions 部署

Todo

### 6.2 GitHub Pages部署

首先在 `mkdocs.yml` 中配置 `repo_url` 和 `repo_name`

```yaml
repo_url: https://github.com/GraythornThanks/GraythornThanks.github.io
repo_name: GraythornThanks/GraythornThanks.github.io
```

然后执行以下命令：

```bash
# 构建站点
mkdocs build

# 部署到GitHub Pages
mkdocs gh-deploy
```

### 6.3 其他部署选项

- Netlify
- Vercel
- 自托管服务器

## 相关资源

- [MkDocs官方文档](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [MkDocs Plugins](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Plugins) 