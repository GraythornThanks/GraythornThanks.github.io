# Graythorn's Blog

这是我的个人博客，使用 MkDocs 构建。

## 本地开发

1. 克隆仓库
```bash
git clone https://github.com/GraythornThanks/GraythornThanks.github.io.git
cd GraythornThanks.github.io
```

2. 创建虚拟环境
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
.\venv\Scripts\activate  # Windows
```

3. 安装依赖
```bash
pip install -r requirements.txt
```

4. 本地运行
```bash
mkdocs serve
```

5. 在浏览器中访问 `http://127.0.0.1:8000`

## 部署

项目通过 GitHub Actions 自动部署到 GitHub Pages。每次推送到 main 分支时都会自动部署。

## 部署状态

[![Deploy to GitHub Pages](https://github.com/GraythornThanks/GraythornThanks.github.io/actions/workflows/deploy.yml/badge.svg)](https://github.com/GraythornThanks/GraythornThanks.github.io/actions/workflows/deploy.yml) 