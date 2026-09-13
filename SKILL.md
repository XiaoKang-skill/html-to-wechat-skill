---
name: html-to-wechat
description: "This skill should be used when converting standard HTML articles (with style blocks, CSS classes, external images, flex layouts) into WeChat editor-compatible HTML. Triggers include requests like 'adapt for WeChat', 'WeChat-compatible HTML', 'inline styles for WeChat', 'convert article for WeChat editor', or any task involving making HTML content compatible with WeChat's rich text editor. The skill handles inline style conversion, base64 image embedding, table colgroup restructuring, and other WeChat-specific constraints."
agent_created: true
---

# HTML to WeChat Converter

## Overview

Convert standard HTML articles into WeChat editor-compatible HTML. WeChat's rich text editor strips `<style>` blocks, external CSS, flexbox, and many modern CSS features. This skill inlines all styles, embeds images as base64, restructures tables with colgroup, and adapts layouts for mobile rendering inside WeChat.

## Workflow

### Step 1: Strip the `<style>` Block — Inline All CSS

1. Remove the entire `<style>...</style>` block from `<head>`.
2. For every element that previously used a CSS class, resolve the class's properties and write them as an inline `style=""` attribute on that element.
3. Keep `<head>` minimal: only `<meta charset>`, `<meta name="viewport">`, and `<title>`.
4. Move body-level styles (`font-family`, `font-size`, `line-height`, `color`, `background`, `margin`, `padding`, `word-wrap`, `overflow-wrap`, `-webkit-text-size-adjust`) directly onto the `<body>` tag as inline styles.

**Viewport meta for WeChat:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

### Step 2: Convert Images to Base64 Inline

1. Find every `<img>` tag with an external `src` (URL or relative path).
2. Read the image file, encode it as base64, and replace the `src` attribute:
   ```html
   <img src="data:image/png;base64,iVBORw0KGgo..." style="width:100%;border-radius:8px;display:block;" />
   ```
3. Always set `width:100%` on images so they fill the mobile content area.
4. Always set `border-radius:8px` for natural rounded corners on mobile.
5. Set `display:block` to avoid inline spacing gaps.
6. Never leave an external image URL — WeChat's editor may block or strip external resources.

**Format mapping:**
- `.png` → `data:image/png;base64,...`
- `.jpg` / `.jpeg` → `data:image/jpeg;base64,...`
- `.gif` → `data:image/gif;base64,...`
- `.webp` → `data:image/webp;base64,...`

### Step 3: Set Outer Wrapper Padding

Use a `<section>` (or `<div>`) as the outermost content wrapper with:
```html
<section style="padding:0 16px 8px;">
  ... article content ...
</section>
```

This matches the standard WeChat content padding: 0 top, 16px horizontal, 8px bottom. If the article needs more breathing room at top/bottom, adjust only the top and bottom values — never reduce the 16px horizontal padding below 16px, as it is the minimum safe margin on most phones.

### Step 4: Restructure Tables with colgroup

WeChat tables need explicit column width control. Apply the following to every `<table>`:

#### 4a. Table-level attributes

```html
<table style="width:100%;table-layout:fixed;border-collapse:collapse;border-style:none;border-width:0;border-color:transparent;margin:16px 0;font-size:13px;"
       cellpadding="0" cellspacing="0" frame="void" rules="none" border="0">
```

Key properties:
- `table-layout:fixed` — force the browser to respect the specified column widths instead of auto-sizing.
- `border-collapse:collapse` — merge cell borders.
- `border-style:none;border-width:0;border-color:transparent` — suppress default borders (WeChat adds unwanted borders).
- `cellpadding="0" cellspacing="0" frame="void" rules="none" border="0"` — HTML attributes as belt-and-suspenders fallback for editors that strip CSS border properties.

#### 4b. Add `<colgroup>` with percentage column widths

Analyze the source HTML to determine each column's appropriate width **as a percentage**, then write those percentages into a `<colgroup>`. All column percentages must total exactly 100%:

```html
<table style="width:100%;table-layout:fixed;..." cellpadding="0" cellspacing="0" frame="void" rules="none" border="0">
<colgroup>
  <!-- percentages must total exactly 100% -->
  <col style="width:16%;">
  <col style="width:21%;">
  <col style="width:21%;">
  <col style="width:21%;">
  <col style="width:21%;">
</colgroup>
<tbody>
  ...
</tbody>
</table>
```

How to determine the percentages:
- If the source CSS already specifies percentage widths (e.g., `th { width:25% }`), carry them over directly and normalize so they total exactly 100%.
- If the source CSS specifies pixel widths, convert to percentages of the total (e.g., `120px + 4×160px = 760px` → `16% / 21% / 21% / 21% / 21%`).
- If no explicit widths exist, infer from content: label/name columns are narrower, data/content columns are wider. Typical split: label column ~24-30% for 2-3 column tables (~16-24% for 4-5 column tables); the remaining data columns share the rest equally.
- Round to whole percentages, then adjust the last column so the total is exactly 100%. Keep label/name columns narrower than data columns.

#### 4c. Write the column percentage into EVERY cell's code

Every `<td>` and `<th>` must repeat its column's percentage inside its own code — both as the inline style `width:X%` AND as the HTML attribute `width="X%"`. This makes each cell self-contained: even if WeChat's editor strips `<colgroup>` or ignores `table-layout:fixed`, every cell still declares its own width, so column proportions survive editing and re-copying inside WeChat.

```html
<!-- column 1 cells -->
<td width="16%" style="width:16%;...">label</td>
<!-- column 2 cells -->
<td width="21%" style="width:21%;...">value</td>
```

The percentage on a cell must exactly match its corresponding `<col>` percentage — never mismatch, never omit it from any cell. This layered approach (colgroup + `table-layout:fixed` + inline `width:X%` on every cell + HTML `width` attribute) ensures consistent rendering across all WeChat clients.

#### 4d. Write the background color into EVERY cell's code

Every `<td>` and `<th>` must also carry an explicit `background` in its own inline style. Never rely on the `<table>`, a `<tr>`, or a parent `<section>` for cell backgrounds — WeChat's editor only reliably keeps what sits directly on the cell:

- Header cells → the header background, e.g. `background:#1E1B4B`.
- Zebra / even-row cells → `background:#fafafa`.
- Normal body cells → `background:#ffffff` — write it explicitly even when the cell "has no background", so the editor can never inject a default gray.
- Cells laid on a colored/gradient card → `background:transparent` so the parent section's background shows through.

Full cell template (width and background always come first, then the visual styles):

```html
<td width="16%" style="width:16%;background:#fafafa;text-align:left;font-weight:600;color:#7C3AED;padding:8px 6px;line-height:1.6;font-size:12px;word-break:break-word;border-style:none;border-width:0;border-bottom:1px solid #eee;">核心目标</td>
```

Every cell also keeps:
- `word-break:break-word` — allow long text to wrap within the cell.
- `border-style:none;border-width:0` — suppress default cell borders (redundant with table attributes but ensures coverage). Declare any visible border (e.g. `border-bottom:1px solid #eee`, or a full grid `border:1px solid #fbcfe8`) AFTER these two.
- `padding`, `text-align`, `font-size`, `line-height`, `color` — carried over from the original CSS class.

Common defects to fix when processing already-half-converted HTML:
- **Duplicate `style` attributes on one element** (e.g. `style="background:..." style="padding:..."`) — browsers drop the second one. Always merge into a single `style` attribute.
- **Row-level `background` on `<tr>`** — move it onto every cell in that row.
- **`<thead>` sections** — merge into `<tbody>` so all rows live in one predictable container.

### Step 5: Replace Unsupported Layouts

#### 5a. Flexbox → Table

WeChat does not reliably support `display:flex`. Replace any flex-based layout (e.g., label-value rows with `justify-content:space-between`) with a `<table>`:

**Before (flex):**
```html
<div style="display:flex;justify-content:space-between;">
  <span>Anthropic Claude布道师</span>
  <span>$240K - $315K</span>
</div>
```

**After (table):**
```html
<table style="width:100%;table-layout:fixed;border-collapse:collapse;border-style:none;border-width:0;" cellpadding="0" cellspacing="0" frame="void" rules="none" border="0">
<colgroup>
  <col style="width:60%;">
  <col style="width:40%;">
</colgroup>
<tbody>
<tr>
<td width="60%" style="width:60%;background:transparent;color:#fff;padding:6px 0;word-break:break-word;border-style:none;border-width:0;">Anthropic Claude布道师</td>
<td width="40%" style="width:40%;background:transparent;color:#4ADE80;font-weight:700;text-align:right;padding:6px 0;word-break:break-word;border-style:none;border-width:0;">$240K - $315K</td>
</tr>
</tbody>
</table>
```

Layout tables follow the same rules as data tables: percentage `<colgroup>` totaling 100%, the column percentage written into every cell's code, and an explicit `background` on every cell (`background:transparent` when the table sits on a colored/gradient card so the parent background shows through).

#### 5b. `<blockquote>` → `<section>`

WeChat may strip `<blockquote>` styling. Convert to `<section>` with the same visual inline styles:

```html
<section style="background:#f7f9fc;border-left:3px solid #7C3AED;padding:14px 16px;margin:16px 0;font-size:14px;color:#555;border-radius:0 6px 6px 0;">quote text</section>
```

#### 5c. `<ul>` / `<ol>` lists → `<p>` with bullet spans

WeChat's list rendering is inconsistent. Convert list items to `<p>` tags with a colored bullet `<span>`:

```html
<p style="padding:8px 0;line-height:1.7;font-size:15px;color:#333;margin:0 0 4px;">
  <span style="color:#7C3AED;font-size:14px;margin-right:6px;">▸</span>
  <strong>item title</strong> — description text
</p>
```

#### 5d. Centered dividers → `<p>` tag

Any `<div>` used purely as a centered section divider (e.g., `✦ ✦ ✦` separator lines) must be converted to `<p>`:

```html
<p style="text-align:center;margin:24px 0;color:#ddd;font-size:12px;letter-spacing:4px;">✦ ✦ ✦</p>
```

WeChat handles `<p>` more consistently than `<div>` for centered inline text. Always use `<p>` for centered separators, divider symbols, or any short text that needs `text-align:center`.

#### 5e. `<div>` semantic blocks → `<section>`

Replace `<div>` wrapper blocks (data boxes, cards, action sections) with `<section>` tags carrying full inline styles. This is semantically cleaner and renders consistently in WeChat.

### Step 6: Body-level Cleanup

1. Set all body styles inline on the `<body>` tag.
2. Ensure every text element has an explicit `color` and `font-size` in its inline style.
3. Ensure every `<p>` has `margin:0 0 16px;line-height:1.8;font-size:16px;color:#333;` unless overridden.
4. Ensure every `<h2>` / `<h3>` has full inline styles including `margin`, `padding`, `border-left`, `font-size`, `font-weight`, `color`, `line-height`.
5. Remove any remaining `class=""` attributes — they have no corresponding CSS anymore.

## Quick Reference: Before → After Checklist

| Element | Standard HTML | WeChat HTML |
|---------|--------------|-------------|
| Styles | `<style>` block in `<head>` | All inline `style=""` on each element |
| Images | `<img src="https://...">` | `<img src="data:image/png;base64,..." style="width:100%;border-radius:8px;display:block;">` |
| Outer wrapper | `<div class="article-wrap">` | `<section style="padding:0 16px 8px;">` |
| Tables | `<table class="compare-table">` | `<table style="width:100%;table-layout:fixed;..." cellpadding="0" cellspacing="0" border="0" frame="void" rules="none">` with `<colgroup>` |
| Column widths | CSS only | `<colgroup><col style="width:X%;">` totaling 100% + `width:X%` inline style + `width="X%"` attribute on every cell |
| Cell backgrounds | row-level CSS / `:nth-child` zebra | explicit `background:...` inline on every single `<td>`/`<th>` (header color / zebra / `#ffffff` / `transparent`) |
| Flex layouts | `display:flex;justify-content:space-between` | `<table>` with two `<td>` columns |
| Blockquotes | `<blockquote>` | `<section>` with inline styles |
| Lists | `<ul><li>` with `:before` pseudo-elements | `<p>` with `<span>` bullet characters |
| Centered dividers | `<div style="text-align:center">` | `<p style="text-align:center;margin:24px 0;color:#ddd;font-size:12px;letter-spacing:4px;">` |
| Semantic blocks | `<div class="card">` | `<section style="...">` |

## Resources

### references/
- `wechat-transform-examples.md` — Detailed before/after transformation examples for each element type, extracted from real article conversions.
