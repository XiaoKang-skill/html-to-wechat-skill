# html-to-wechat

WorkBuddy Skill — 将标准 HTML 文章转换为微信公众号编辑器兼容的 HTML 格式。

## 功能

- 剥离 `<style>` 块，所有 CSS 内联化
- 图片转 base64 内嵌
- 表格重构：colgroup 列百分比（合计 100%）+ table-layout:fixed + 每个单元格写入 `width:X%`（内联样式 + `width` 属性）+ 每个单元格显式内联背景色
- Flexbox → Table 布局替换
- blockquote / ul / ol / div 等元素适配微信渲染
- 外层 wrapper padding 适配移动端

## 文件结构

```
html-to-wechat/
├── SKILL.md                              # Skill 主文件（完整转换工作流）
├── references/
│   └── wechat-transform-examples.md      # 各元素类型的 before/after 转换示例
└── .gitignore
```

## 使用方式

在 [WorkBuddy](https://www.codebuddy.cn/) 中通过 `@skill:html-to-wechat` 调用，传入 HTML 文件路径即可自动执行转换。

## 适用场景

- 将带有外部 CSS、flex 布局的 HTML 文章转为微信公众号可直接粘贴的格式
- 确保表格列宽、图片、排版在微信编辑器中一致渲染
