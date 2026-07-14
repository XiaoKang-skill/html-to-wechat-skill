# html-to-wechat

WorkBuddy Skill — 将标准 HTML 文章转换为微信公众号编辑器兼容的 HTML。

## 功能

- 移除 `<style>` 块，将所有 CSS 转为内联样式
- 将外部图片转为 base64 内嵌
- 用 `<section>` + 内联样式替换 flex 布局
- 用 `<table>` + `<colgroup>` 重构表格，确保微信端列宽一致
- 将 `<blockquote>` / `<ul>` / `<ol>` 等转为微信兼容的 `<section>` / `<p>` 标签

## 使用

将此 skill 安装到 WorkBuddy 的 skills 目录即可使用。当你在对话中提到"适配微信"、"微信公众号格式"等关键词时，该 skill 会自动触发。

## 结构

```
html-to-wechat-skill/
├── SKILL.md                          # Skill 定义与完整工作流程
├── README.md                          # 本文件
└── references/
    └── wechat-transform-examples.md   # 详细前后对比示例
```

## 许可

MIT License
