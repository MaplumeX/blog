# AGENTS.md

给自动化/代理式编码工具使用的仓库指南。目标：让代理能在不猜测的前提下，
快速跑通 build/lint/check，并遵循现有代码风格。

## 项目概览

- 技术栈：Astro + TypeScript + Tailwind CSS + Svelte（Astro islands）
- 包管理器：pnpm（`package.json` 有 `preinstall` 限制，仅允许 pnpm）
- 代码质量/格式化：Biome（仓库根有 `biome.json`；npm scripts 仅作用于 `./src`）

## 常用命令（从仓库根目录执行）

安装依赖：

```bash
pnpm install
```

提示：

- `preinstall` 会强制使用 pnpm（不要用 npm/yarn/bun）。
- `.npmrc` 开启 `manage-package-manager-versions`；尽量保持 `package.json#packageManager` 的版本一致。

开发/预览：

```bash
pnpm dev        # astro dev
pnpm preview    # astro preview（需先 build）
```

构建：

```bash
pnpm build      # astro build && pagefind --site dist
pnpm astro build
```

说明：

- `pnpm build` 会额外跑 Pagefind 索引；只想复现 CI 的 build 用 `pnpm astro build`
  （CI 在 `.github/workflows/build.yml` 里用的就是 `pnpm astro build`）。

检查（推荐作为“测试替代”）：

```bash
pnpm check        # astro check
pnpm astro check
pnpm type-check   # tsc --noEmit --isolatedDeclarations
```

格式化 / Lint（Biome）：

```bash
pnpm format     # biome format --write ./src
pnpm lint       # biome check  --write ./src（会改文件）
```

新建文章：

```bash
pnpm new-post <filename>
```

## 只跑“单个文件”的快速命令

注意：`pnpm format`/`pnpm lint` 固定作用于 `./src`。要对单文件执行，直接调用 Biome/tsc：

```bash
pnpm exec biome format --write src/path/to/file.ts
pnpm exec biome check --write src/path/to/file.ts
pnpm exec biome ci src/path/to/file.ts

pnpm exec tsc --noEmit --isolatedDeclarations src/path/to/file.ts
```

说明：

- 单文件 `tsc` 更适合做快速探测；涉及路径别名/生成类型（Astro）时优先跑 `pnpm type-check` / `pnpm check`。

## 测试（Tests）现状

- 当前仓库没有配置可运行的测试框架/脚本（`package.json` 无 `test` script，也未引入 Vitest/Jest/Playwright）。
- 实际质量门禁是：`pnpm check` / `pnpm type-check` / `pnpm build`，以及 CI 的 `biome ci`。

## 代码风格（以现有配置/代码为准）

Biome（`biome.json`）：

- 缩进：tab
- 字符串：双引号
- 自动整理 imports：开启（在 VS Code 保存时也会触发）
- `pnpm lint` 会 `--write` 自动修复；提交前检查一下 diff
- 注意：Biome 配置排除 `src/**/*.css`、`dist/`、`node_modules/` 等；改 CSS 时保持现有风格即可
- `.astro`/`.svelte` 有 linter overrides：未使用 import/变量等规则在这些文件里被关闭，避免依赖它“容忍垃圾导入”

TypeScript（`tsconfig.json`）：

- 严格模式（extends `astro/tsconfigs/strict`）
- 路径别名（优先使用）：
  - `@components/*`, `@assets/*`, `@constants/*`, `@utils/*`, `@i18n/*`, `@layouts/*`, `@/*`

Imports：

- 类型用 `import type { ... }` / `import { type Foo } ...`（仓库里大量使用）
- import 路径是否带 `.ts` 扩展名在仓库内不完全一致：保持“同文件/同目录”的既有风格，不要在一次改动里混用

命名与文件组织：

- 组件文件：`PascalCase`（如 `src/components/Navbar.astro`）
- 工具模块：`kebab-case`（如 `src/utils/setting-utils.ts`）
- 常量：`UPPER_SNAKE_CASE`（如 `src/constants/constants.ts`）

错误处理与健壮性：

- 避免空 `catch`；需要记录时用 `console.error` 并带上下文
- 多用 early return，减少深层嵌套
- API 路由（`src/pages/*.ts`）返回 `Response` 时显式设置 `Content-Type`（参考 `src/pages/robots.txt.ts`）
- 若必须忽略 Biome 规则，使用最小范围的 `// biome-ignore ...: <reason>` 并写明原因（参考 `src/pages/rss.xml.ts`）

## Astro / Svelte 约定

- Astro 组件可渲染 Svelte islands（见 `src/components/*.svelte` 的 `client:only="svelte"` 用法）
- Svelte 为 v5（runes）：代码里使用 `$state(...)` + `onMount(...)` 管理状态/副作用
- 注意 server/client 边界：部分逻辑用 `import.meta.env.PROD` 做生产环境 gating

## CI / 工作流

- 代码质量：`.github/workflows/biome.yml` 跑 `biome ci ./src --reporter=github`
- 构建与检查：`.github/workflows/build.yml` 跑 `pnpm astro check` 与 `pnpm astro build`

## 编辑器（VS Code）

- `/.vscode/settings.json`：默认 formatter 为 Biome，保存时自动 fix + organize imports
- `/.vscode/extensions.json`：推荐安装 Biome 与 Astro 插件

## Cursor / Copilot 规则

- 未发现 Cursor 规则（`.cursor/rules/` 或 `.cursorrules` 不存在）
- 未发现 Copilot 指令（`.github/copilot-instructions.md` 不存在）

## 提交/协作

- `CONTRIBUTING.md` 建议使用 Conventional Commits，并在提交前至少运行：

```bash
pnpm check
pnpm format
```
