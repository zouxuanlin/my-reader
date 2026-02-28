# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在此仓库中工作提供指导。

## 项目概述

小说阅读网页应用，使用 Vue 3 + Vite + Tailwind CSS 构建。

### 技术栈
- **Vue 3** - 使用 Composition API (`<script setup>`)
- **Vite** - 构建工具和开发服务器
- **Tailwind CSS v4** - 使用 `@tailwindcss/vite` 插件

### 常用命令

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览构建结果
npm run preview
```

## 目录结构

```
src/
├── components/
│   └── NovelReader.vue    # 主阅读器组件（包含所有功能）
├── data/
│   └── novel.js           # 小说示例数据
├── App.vue                # 应用入口
├── main.js                # 应用入口点
└── style.css              # 全局样式（含 Tailwind 导入）
```

## 组件架构

**NovelReader.vue** 是核心组件，包含：
- 章节列表侧边栏
- 阅读器主界面
- 设置面板（字体大小、背景主题）
- 键盘导航（左右箭头切换章节）
- localStorage 持久化阅读进度

## 数据格式

小说数据位于 `src/data/novel.js`，格式如下：

```javascript
{
  title: "小说标题",
  author: "作者",
  chapters: [
    { id: 1, title: "章节标题", content: "章节内容..." }
  ]
}
```

## 功能特性

1. **阅读设置**：4 种背景主题（白、米色、 sepia、暗黑），字体大小 14-28px 可调
2. **进度保存**：自动保存章节、滚动位置、设置到 localStorage
3. **导航**：上一章/下一章按钮，键盘左右箭头，章节列表
4. **响应式设计**：适配移动端和桌面端
