# MuyuQ.github.io

个人博客站点，基于 [Zensical](https://zensical.org/) 静态站点框架构建（Python + uv 管理），发布在 GitHub Pages。

> 本仓库对应远端 `MuyuQ/Zensical_blog`，站点内容通过 GitHub Pages 以 `muyuq.github.io` 域名对外提供。

## 目录结构

- `docs/`：Markdown 源文件，按主题分目录（`C`、`DB`、`Mac`、`Python`、`杂`、`生活`）
- `site/`：构建产物
- `zensical.toml`：站点配置（站点名、主题、导航与功能开关）
- `pyproject.toml` / `uv.lock`：Python 依赖与锁文件

## 常用命令

```bash
# 本地预览
uv run zensical serve

# 构建到 site/
uv run zensical build
```

---
*最后更新: 2026-09-27*
