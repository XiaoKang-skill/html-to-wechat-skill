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

#### 4b. Add `<colgroup>` with column widths matching the source HTML

Analyze the source HTML to determine each column's proportional width, then use `<colgroup>` with `<col>` elements reflecting those proportions:

```html
<table style="width:100%;table-layout:fixed;..." cellpadding="0" cellspacing="0" frame="void" rules="none" border="0">
<colgroup>
  <!-- pixel values reflect the ratio derived from the source HTML -->
  <col style="width:120px;">
  <col style="width:160px;">
  ...
</colgroup>
<tbody>
  ...
</tbody>
</table>
```

How to determine proportions from the source HTML:
- If the source CSS specifies column widths (e.g., `td:first-child { width:120px }`, `th { width:25% }`), carry those values over directly.
- If the source uses percentage-based widths, convert to pixel proportion hints (e.g., `25% / 75%` → `120 / 360` or `1 / 3` ratio → `120 / 360`).
- If no explicit widths exist, infer from content: label/name columns are narrower, data/content columns are wider. Typical ratios: label column ~30%, data columns ~70% split equally.
- The pixel values are proportional hints — `table-layout:fixed` distributes available width according to the ratio. Keep label/name columns narrower than data columns.

#### 4c. Add `width` attribute on every `<td>` matching colgroup

Even with colgroup and table-layout:fixed, some WeChat rendering paths ignore colgroup. To guarantee column widths, add the `width` HTML attribute on every `<td>` and `<th>`, using the same pixel values as the corresponding `<col>`:

```html
<!-- width values must match the colgroup proportions -->
<td width="120" style="...">label</td>
<td width="160" style="...">value</td>
```

This triple-layered approach (colgroup + table-layout:fixed + td width attribute) ensures consistent rendering across all WeChat clients. The `width` attribute on `<td>` must use the same value as its corresponding `<col>` element — never mismatch.

#### 4d. Cell-level inline styles

Every `<td>` and `<th>` must carry:
- `word-break:break-word` — allow long text to wrap within the cell.
- `border-style:none;border-width:0` — suppress default cell borders (redundant with table attributes but ensures coverage).
- `padding`, `text-align`, `font-size`, `line-height`, `color`, `background` — carried over from the original CSS class.

For zebra striping, set `background:#fafafa` on even-row cells directly in the inline style.

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
<tbody>
<tr>
<td style="color:#fff;padding:6px 0;word-break:break-word;border-style:none;border-width:0;">Anthropic Claude布道师</td>
<td style="color:#4ADE80;font-weight:700;text-align:right;padding:6px 0;word-break:break-word;border-style:none;border-width:0;">$240K - $315K</td>
</tr>
</tbody>
</table>
```

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
| Column widths | CSS only | `<colgroup><col style="width:Xpx;">` + `<td width="X">` (proportions from source HTML) |
| Flex layouts | `display:flex;justify-content:space-between` | `<table>` with two `<td>` columns |
| Blockquotes | `<blockquote>` | `<section>` with inline styles |
| Lists | `<ul><li>` with `:before` pseudo-elements | `<p>` with `<span>` bullet characters |
| Centered dividers | `<div style="text-align:center">` | `<p style="text-align:center;margin:24px 0;color:#ddd;font-size:12px;letter-spacing:4px;">` |
| Semantic blocks | `<div class="card">` | `<section style="...">` |

## Resources

### references/
- `wechat-transform-examples.md` — Detailed before/after transformation examples for each element type, extracted from real article conversions.
