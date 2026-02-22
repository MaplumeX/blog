# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概览

基于 [Fuwari](https://github.com/saicaca/fuwari) 模板的静态博客，技术栈：**Astro 5 + TypeScript (strict) + Tailwind CSS 3 + Svelte 5 (islands)**。包管理器强制使用 **pnpm**（`preinstall` 脚本校验）。

## 常用命令

```bash
pnpm install              # 安装依赖（禁止 npm/yarn/bun）
pnpm dev                  # 本地开发 localhost:4321
pnpm build                # 生产构建 + Pagefind 搜索索引
pnpm preview              # 预览生产构建（需先 build）
pnpm check                # Astro 类型检查
pnpm type-check           # tsc --noEmit --isolatedDeclarations
pnpm format               # Biome 格式化 ./src
pnpm lint                 # Biome 检查+自动修复 ./src
pnpm new-post <filename>  # 创建新博文模板
```

单文件检查：
```bash
pnpm exec biome check --write src/path/to/file.ts
pnpm exec biome format --write src/path/to/file.ts
```

**无测试框架**。质量门禁依赖 `pnpm check` + `pnpm type-check` + `pnpm build` + CI `biome ci`。

## 架构

### 内容管理

Astro Content Collections，博文在 `src/content/posts/`，支持单文件或 bundle 目录（含本地图片）：
```
src/content/posts/
├── my-post.md                 # 单文件
└── post-with-images/
    ├── index.md               # bundle 形式
    └── cover.jpg
```

Frontmatter 必填 `title` + `published`，可选 `description`, `image`, `tags[]`, `category`, `draft`, `lang`, `updated`。Schema 定义在 `src/content/config.ts`。

### 路由 (文件路由)

- `/` → `src/pages/[...page].astro` (分页首页)
- `/posts/[slug]/` → `src/pages/posts/[...slug].astro`
- `/about/`, `/archive/` → 对应 `.astro` 页面
- `/rss.xml`, `/robots.txt` → TypeScript 端点

### 组件边界

- **Astro 组件** (.astro)：静态渲染，处理布局和数据获取
- **Svelte 组件** (.svelte)：交互式岛屿，通过 `client:only="svelte"` 挂载。Svelte v5 runes 语法（`$state`, `$derived` 等）
- `import.meta.env.PROD` 控制生产/开发环境差异（如 Pagefind 搜索仅生产环境可用）

### Markdown 扩展

通过 `astro.config.mjs` 中的 Remark/Rehype 插件链：
- 数学公式 (KaTeX)、阅读时间、自动摘要
- Admonitions (`:::note`, `:::tip`, `> [!WARNING]` 等)
- GitHub 仓库卡片 (`::github{repo="owner/repo"}`)
- Expressive Code 代码块（行号、折叠、语言标签）

自定义插件位于 `src/plugins/`。

### 样式系统

- Tailwind + Stylus 预处理器
- 主题色基于 oklch 色彩空间，通过 CSS 变量 `--hue` 动态调整
- 深浅模式：class-based (`dark` class)
- 全局样式变量定义在 `src/styles/variables.styl`
- Tailwind 组件层类定义在 `src/styles/main.css`

### 国际化

`src/i18n/` 支持 11 种语言，通过 `i18n(I18nKey.xxx)` 函数调用。站点语言在 `src/config.ts` 的 `siteConfig.lang` 配置。

### 页面转场与交互

Swup 库实现无刷新页面转换（`@swup/astro`），OverlayScrollbars 自定义滚动条，PhotoSwipe 图片灯箱。

## 代码风格

- **Biome** 统管格式化和 lint（`biome.json`）：tab 缩进、双引号、自动 import 排序
- Biome 排除 CSS 文件；`.astro`/`.svelte` 文件关闭未使用导入/变量规则
- 类型导入用 `import type { ... }`
- **路径别名优先**：`@components/*`, `@utils/*`, `@assets/*`, `@constants/*`, `@i18n/*`, `@layouts/*`, `@/*`
- 命名：组件 `PascalCase`，工具模块 `kebab-case`，常量 `UPPER_SNAKE_CASE`
- 避免空 catch，多用 early return，API 路由显式设置 Content-Type
- Biome 忽略注解：`// biome-ignore <rule>: <reason>`
- Conventional Commits 格式提交

## 站点配置入口

`src/config.ts` 是用户修改的主入口，包含站点标题、主题色 hue、导航栏、个人资料、许可证等配置。类型定义在 `src/types/config.ts`。

## MCP
需要修改astro文件时，先调用astro-docs MCP获取相关文档

## CI

- `.github/workflows/biome.yml`：`biome ci ./src`
- `.github/workflows/build.yml`：`pnpm astro check` + `pnpm astro build`（Node 22/23）
