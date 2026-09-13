# WeChat HTML Transformation Examples

Detailed before/after examples for each element type, extracted from real article conversions.

## 1. Document Structure

### Before (Standard HTML)

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Article Title</title>
<style>
* { margin:0; padding:0; box-sizing:border-box; }
body {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", sans-serif;
  font-size: 16px;
  line-height: 1.8;
  color: #333;
  background: #fff;
  max-width: 100%;
  -webkit-text-size-adjust: 100%;
}
.article-wrap { padding: 20px 16px 40px; }
/* ... more classes ... */
</style>
</head>
<body>
<div class="article-wrap">
  ...
</div>
</body>
</html>
```

### After (WeChat HTML)

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Article Title</title>
</head>
<body style="font-family:-apple-system,BlinkMacSystemFont,'PingFang SC','Hiragino Sans GB','Microsoft YaHei',sans-serif;font-size:16px;line-height:1.8;color:#333;background:#fff;margin:0;padding:0;word-wrap:break-word;overflow-wrap:break-word;-webkit-text-size-adjust:100%;">

<section style="padding:0 16px 8px;">
  ...
</section>
</body>
</html>
```

Key changes:
- `<style>` block removed entirely
- All body styles moved to `<body style="">`
- `box-sizing` removed (WeChat ignores it)
- `word-wrap:break-word;overflow-wrap:break-word` added to body for text wrapping safety
- `maximum-scale=1.0, user-scalable=no` added to viewport to prevent zoom issues
- Outer wrapper changed from `<div class="article-wrap">` to `<section style="padding:0 16px 8px;">`

---

## 2. Images

### Before

```html
<img src="https://example.com/images/chart.png" alt="Salary chart" class="article-image">
```

### After

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA..." alt="Salary chart" style="width:100%;border-radius:8px;display:block;margin:16px 0;" />
```

Rules:
- `src` must be `data:image/png;base64,...` (or jpeg/gif/webp as appropriate)
- `width:100%` — fill mobile content area
- `border-radius:8px` — natural rounded corners
- `display:block` — eliminate inline image gap
- No external URL references

---

## 3. Tables (5-column comparison table)

### Before

```html
<table class="compare-table">
<tr><th>维度</th><th>AI布道师</th><th>公关/传播总监</th><th>明星代言人</th><th>技术售前</th></tr>
<tr><td>核心目标</td><td>赢得开发者信任</td><td>维护品牌形象</td><td>扩大知名度</td><td>促成签单</td></tr>
<tr><td>是否写代码</td><td>是，能现场写Demo</td><td>否</td><td>否</td><td>少量或不写</td></tr>
</table>
```

CSS was:
```css
.compare-table { width:100%; border-collapse:collapse; margin:16px 0; font-size:13px; }
.compare-table th { background:#1E1B4B; color:#fff; padding:8px 6px; text-align:center; font-weight:600; font-size:12px; }
.compare-table td { padding:8px 6px; border-bottom:1px solid #eee; text-align:center; line-height:1.6; font-size:12px; }
.compare-table tr:nth-child(even) td { background:#fafafa; }
.compare-table td:first-child { text-align:left; font-weight:600; color:#7C3AED; }
```

### After

```html
<table style="width:100%;table-layout:fixed;border-collapse:collapse;border-style:none;border-width:0;border-color:transparent;margin:16px 0;font-size:13px;" cellpadding="0" cellspacing="0" frame="void" rules="none" border="0">
<colgroup>
  <col style="width:16%;">
  <col style="width:21%;">
  <col style="width:21%;">
  <col style="width:21%;">
  <col style="width:21%;">
</colgroup>
<tbody>
<tr>
  <th width="16%" style="width:16%;background:#1E1B4B;color:#fff;padding:8px 6px;text-align:center;font-weight:600;font-size:12px;word-break:break-word;border-style:none;border-width:0;">维度</th>
  <th width="21%" style="width:21%;background:#1E1B4B;color:#fff;padding:8px 6px;text-align:center;font-weight:600;font-size:12px;word-break:break-word;border-style:none;border-width:0;">AI布道师</th>
  <th width="21%" style="width:21%;background:#1E1B4B;color:#fff;padding:8px 6px;text-align:center;font-weight:600;font-size:12px;word-break:break-word;border-style:none;border-width:0;">公关/传播总监</th>
  <th width="21%" style="width:21%;background:#1E1B4B;color:#fff;padding:8px 6px;text-align:center;font-weight:600;font-size:12px;word-break:break-word;border-style:none;border-width:0;">明星代言人</th>
  <th width="21%" style="width:21%;background:#1E1B4B;color:#fff;padding:8px 6px;text-align:center;font-weight:600;font-size:12px;word-break:break-word;border-style:none;border-width:0;">技术售前</th>
</tr>
<tr>
  <td width="16%" style="width:16%;background:#fafafa;text-align:left;font-weight:600;color:#7C3AED;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">核心目标</td>
  <td width="21%" style="width:21%;background:#fafafa;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">赢得开发者信任</td>
  <td width="21%" style="width:21%;background:#fafafa;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">维护品牌形象</td>
  <td width="21%" style="width:21%;background:#fafafa;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">扩大知名度</td>
  <td width="21%" style="width:21%;background:#fafafa;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">促成签单</td>
</tr>
<tr>
  <td width="16%" style="width:16%;background:#ffffff;text-align:left;font-weight:600;color:#7C3AED;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">是否写代码</td>
  <td width="21%" style="width:21%;background:#ffffff;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">是，能现场写Demo</td>
  <td width="21%" style="width:21%;background:#ffffff;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">否</td>
  <td width="21%" style="width:21%;background:#ffffff;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">否</td>
  <td width="21%" style="width:21%;background:#ffffff;text-align:center;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">少量或不写</td>
</tr>
</tbody>
</table>
```

Key changes:
- `<colgroup>` added with 5 `<col>` elements using percentage widths (16% + 4×21% = 100%)
- `table-layout:fixed` forces these widths
- Every `<td>` / `<th>` repeats its column percentage in its own code: `width="16%"` / `width="21%"` HTML attribute + `width:16%` / `width:21%` inline style — each cell is self-contained even if colgroup is stripped
- Every cell carries an explicit inline `background`: header cells `#1E1B4B`, zebra (even) rows `#fafafa`, normal rows `#ffffff` — backgrounds are never inherited from the table or row
- `border-style:none;border-width:0` on every cell, with the visible `border-bottom:1px solid #eee` declared after them
- `word-break:break-word` on every cell for text wrapping safety

---

## 4. Tables (2-column label/value inside a card)

### Before (flex layout)

```html
<div class="salary-card">
  <div class="label">🌍 美国市场</div>
  <div class="row"><span>Anthropic Claude布道师</span><span class="amount">$240K - $315K</span></div>
  <div class="row"><span>Adobe AI布道师</span><span class="amount">$270K+</span></div>
</div>
```

CSS was:
```css
.salary-card { background:linear-gradient(135deg,#1E1B4B,#312E81); color:#fff; border-radius:10px; padding:18px; margin:16px 0; font-size:14px; }
.salary-card .row { display:flex; justify-content:space-between; padding:6px 0; border-bottom:1px solid rgba(255,255,255,0.1); }
.salary-card .amount { color:#4ADE80; font-weight:700; }
```

### After (table-based layout)

```html
<section style="background:linear-gradient(135deg,#1E1B4B,#312E81);color:#fff;border-radius:10px;padding:18px;margin:16px 0;font-size:14px;">
<p style="color:#A78BFA;font-size:12px;margin:0 0 8px;letter-spacing:1px;">🌍 美国市场</p>
<table style="width:100%;table-layout:fixed;border-collapse:collapse;border-style:none;border-width:0;border-color:transparent;" cellpadding="0" cellspacing="0" frame="void" rules="none" border="0">
<colgroup>
  <col style="width:60%;">
  <col style="width:40%;">
</colgroup>
<tbody>
<tr>
  <td width="60%" style="width:60%;background:transparent;color:#fff;padding:6px 0;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid rgba(255,255,255,0.1);">Anthropic Claude布道师</td>
  <td width="40%" style="width:40%;background:transparent;color:#4ADE80;font-weight:700;text-align:right;padding:6px 0;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid rgba(255,255,255,0.1);">$240K - $315K</td>
</tr>
<tr>
  <td width="60%" style="width:60%;background:transparent;color:#fff;padding:6px 0;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid rgba(255,255,255,0.1);">Adobe AI布道师</td>
  <td width="40%" style="width:40%;background:transparent;color:#4ADE80;font-weight:700;text-align:right;padding:6px 0;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid rgba(255,255,255,0.1);">$270K+</td>
</tr>
</tbody>
</table>
</section>
```

Key changes:
- `display:flex` replaced with `<table>` (flex not supported in WeChat)
- Percentage `<colgroup>` (60% / 40%) — label column wider, right-aligned amount column narrower
- Every cell repeats its column percentage in its own code: `width="60%"` / `width="40%"` HTML attribute + `width:60%` / `width:40%` inline style
- Every cell carries an explicit `background:transparent` so the card's gradient shows through
- `.salary-card` div → `<section>` with full inline styles
- `.row` flex div → `<tr>` with two `<td>` columns
- `.amount` class → inline `color:#4ADE80;font-weight:700;text-align:right`

---

## 5. Blockquote

### Before

```html
<blockquote>
  "销售人员会说，给我2500美元，我给你这台电脑。而我们试图传递的是创造力和生产力提升的好消息。"
</blockquote>
```

### After

```html
<section style="background:#f7f9fc;border-left:3px solid #7C3AED;padding:14px 16px;margin:16px 0;font-size:14px;color:#555;border-radius:0 6px 6px 0;">
  "销售人员会说，给我2500美元，我给你这台电脑。而我们试图传递的是创造力和生产力提升的好消息。"
</section>
```

---

## 6. Unordered Lists

### Before

```html
<ul class="point-list">
  <li><strong>至少7年技术创业经验</strong>——这意味着你不是"学过"，而是"干过"</li>
  <li><strong>能设计并开展动手技术课程</strong>——让开发者从好奇到真正用Claude构建产品</li>
</ul>
```

CSS was:
```css
.point-list { list-style:none; padding:0; margin:12px 0; }
.point-list li { padding:8px 0 8px 24px; position:relative; line-height:1.7; font-size:15px; }
.point-list li:before { content:"▸"; position:absolute; left:4px; color:#7C3AED; font-size:12px; }
```

### After

```html
<p style="padding:8px 0;line-height:1.7;font-size:15px;color:#333;margin:0 0 4px;">
  <span style="color:#7C3AED;font-size:14px;margin-right:6px;">▸</span>
  <strong>至少7年技术创业经验</strong>——这意味着你不是"学过"，而是"干过"
</p>
<p style="padding:8px 0;line-height:1.7;font-size:15px;color:#333;margin:0 0 4px;">
  <span style="color:#7C3AED;font-size:14px;margin-right:6px;">▸</span>
  <strong>能设计并开展动手技术课程</strong>——让开发者从好奇到真正用Claude构建产品
</p>
```

Key changes:
- `<ul>` / `<li>` → individual `<p>` tags
- `:before` pseudo-element (not supported in WeChat) → `<span>` with bullet character
- Bullet color and size set inline on the `<span>`

---

## 7. Centered Dividers

### Before

```html
<div style="text-align:center;margin:24px 0;color:#ddd;font-size:12px;letter-spacing:4px;">✦ ✦ ✦</div>
```

### After

```html
<p style="text-align:center;margin:24px 0;color:#ddd;font-size:12px;letter-spacing:4px;">✦ ✦ ✦</p>
```

Key changes:
- `<div>` → `<p>` for centered divider text
- WeChat handles `<p>` more consistently than `<div>` for centered inline text
- Always use `<p>` for centered separators, divider symbols, or any short text that needs `text-align:center`

---

## 8. Headings

### Before

```html
<h2>1983年，苹果发明了这个岗位</h2>
```

CSS was:
```css
h2 { font-size:19px; font-weight:700; color:#1a1a1a; margin:28px 0 12px; padding-left:12px; border-left:3px solid #7C3AED; line-height:1.4; }
```

### After

```html
<h2 style="font-size:19px;font-weight:700;color:#1a1a1a;margin:28px 0 12px;padding-left:12px;border-left:3px solid #7C3AED;line-height:1.4;">1983年，苹果发明了这个岗位</h2>
```

---

## 9. Paragraphs

### Before

```html
<p>5月，Claude母公司Anthropic挂出了一个让人摸不着头脑的职位——<strong>应用AI Claude布道师</strong>。</p>
```

### After

```html
<p style="margin:0 0 16px;line-height:1.8;font-size:16px;color:#333;">5月，Claude母公司Anthropic挂出了一个让人摸不着头脑的职位——<strong>应用AI Claude布道师</strong>。</p>
```

Every `<p>` gets: `margin:0 0 16px;`, `line-height:1.8;`, `font-size:16px;`, `color:#333;` unless explicitly overridden.

---

## 10. Data Box / Callout

### Before

```html
<div class="data-box">
  <strong>📊 一句话看懂：</strong>AI布道师不是销售、不是公关、不是技术代言人。
</div>
```

### After

```html
<section style="background:linear-gradient(135deg,#F5F3FF,#EDE9FE);border-left:4px solid #7C3AED;padding:16px 18px;margin:20px 0;border-radius:0 8px 8px 0;font-size:15px;">
  <strong style="color:#7C3AED;">📊 一句话看懂：</strong>AI布道师不是销售、不是公关、不是技术代言人。
</section>
```

Key changes:
- `<div class="data-box">` → `<section>` with full inline styles
- `<strong>` gets `color:#7C3AED` inline (previously via `.data-box strong` CSS rule)
