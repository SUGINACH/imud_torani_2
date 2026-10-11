# IMUD-TORANI → Postext Configuration Mapping

## 📋 ניתוח מקיף של ההגדרות

### התוכנה שלך (IMUD-TORANI) - רכיבים עיקריים

#### 1. **Shaars (שערים) - System**
```
רשום ב: decorations.js (lines 410-625)

📝 Main Shaar (שער ראשי):
  - renderMainShaar() → pages[1]
  - Box Position: top 20mm, bottom 16mm, right 15mm, left 15mm
  - Content: בס"ד, title, subtitle, SVG divider, description, sections, author, year
  - Theme Selection: select-shaar-main (3 builtin + JSON imports)
  - Background: custom SVG or built-in patterns (m1, m2, m3)

📝 Back Shaar (שער אחורי):
  - renderBackShaar() → pages[verso before main]
  - Box Position: top 22mm, bottom 18mm, right 15mm, left 95mm
  - Content: custom text, title, sections, author, year
  - Theme Selection: select-shaar-back

📝 Sub Shaar (שער משנה):
  - renderSubShaar() → pages[after H1]
  - Box Position: top 20mm, bottom 20mm, right 95mm, left 15mm
  - Content: title (split on dash), subtitle
  - Theme Selection: select-shaar-sub

⚙️ Shaar Configuration Sources:
  - BUILTIN_SHAAR_THEMES: 3 themes (builtin_1, builtin_2, builtin_3)
  - EXTERNAL_SHAAR_ASSETS: JSON imports {main: {}, back: {}, sub: {}}
  - Canvas Editor: drag/resize handles for all shaars
  - Custom builder: inputs for top/bottom/left/right per shaar type
```

**✅ Maps to Postext:**
- `headingStyles["cover"]` with `advancedDesign.slot` → main shaar design
- `headingStyles["back"]` → back shaar
- `headingStyles` with level 1 + `advancedDesign` → sub shaar opener
- Design slots for all visual elements (text, rules, boxes, images)

**⚠️ Gaps:**
- Postext doesn't have "shaar theme" concept per se
- Must construct from design slots + background
- No direct "drag/resize" UI - must export fixed coordinates

---

#### 2. **Decorations System**
```
רשום ב: decorations.js (entire file)

🎨 Ornaments (Left-Right / Up-Down):
  - ORNAMENTS_LR: { diamonds, spear, swash, pen }
  - ORNAMENTS_UD: { diamonds, spear, swash, pen }
  - HEADER_ORNAMENTS: { diamonds, spear, swash, pen }
  - SECTION_DIVIDERS: { pyramid, spear, swash, flourish }
  - NOTE_RULE_GLYPHS: { dots, sparkle, star, diamond }

  Applied in:
  - getHeadingOrnamentHTML(type, isTop, isH2) → top/bottom of H2
  - renderSideOrnamentHTML(styleKey, isLeft, context) → left/right of H3
  - buildNoteRuleElement() → separator lines in footnotes
  - getSectionDividerHTML(type) → dividers between sections

📐 Dimension Controls:
  - dim-h2-top-w / dim-h2-top-h → H2 top ornament size
  - dim-h2-btm-w / dim-h2-btm-h → H2 bottom ornament size
  - dim-h3-side-w / dim-h3-side-h → H3 side ornaments size
  - dim-secdiv-w / dim-secdiv-h → section divider size
  - dim-note-side-w / dim-note-side-h → note ornament size
  - h2-orn-fit-text → auto-width to heading text width

🎯 Heading Ornament Scope:
  - h2-orn-scope: 'both' | 'siman_only' | 'title_only'
  - Controls whether ornaments wrap heading + siman or just one

🔄 Custom Imports:
  - registerExternalShaarAsset(typeKey, id, name, svgContent)
  - registerOrnamentsLR(id, name, svgContent)
  - registerOrnamentsUD(id, name, svgContent)
  - registerExternalAllOrnament(id, svgContent)
  - registerExternalHeader(id, svgContent)
  - registerExternalDivider(id, svgContent)
  - registerExternalNoteRule(id, svgContent)
```

**✅ Maps to Postext:**
- Ornaments → `headingAdvancedDesign.slot.elements[]` as decorative boxes/text/images
- Section dividers → `heading.levels[2].advancedDesign.slot` or custom design slot
- Note rules → `footnotes.separatorConfig` (limited support)
- Dimension controls → `ElementPlacement.size` in design slot
- Custom SVG → base64 encode in `DesignImageElement.resourceId` or inline as `DesignBoxElement`

**⚠️ Gaps:**
- Postext lacks ornament library concept
- Must hand-code each ornament in design slot
- No "fit to text width" auto-sizing (must calculate width)
- Limited footnote rule styling vs. IMUD's flexibility

---

#### 3. **Fonts System**
```
רשום ב: app.js (font-picker selects) + style.css (CSS variables)

🔤 Font Configuration:
  - f-sh-title, s-sh-title → Shaar book title (family + size)
  - f-sh-sub, s-sh-sub → Shaar subtitle
  - f-sh-desc, s-sh-desc → Shaar description
  - f-sh-sec, s-sh-sec → Shaar section/edition info
  - f-sh-author, s-sh-author → Shaar author name
  - f-sh-year, s-sh-year → Shaar year/place
  - font-body → body text family
  - size-body → body text size (pt)
  - line-height-body → body leading (value, not ratio)
  - h2-ornament-* → H2 ornament styling
  - h3-ornament-* → H3 ornament styling

📦 Default fonts (style.css):
  --fn-h0: 'David Libre'
  --fn-h1: 'Bellefair'
  --fn-h2: 'David Libre'
  --fn-h3: 'Noto Serif Hebrew'
  --fn-body: 'Frank Ruhl Libre'
  --fn-dh: 'David Libre' (Dibur Hamatchil)
  --fn-note-title: 'David Libre'

🎨 System fonts loader:
  loadSystemFonts() → queries user's computer fonts
  loadExternalFonts() → from Google Fonts / Fontsource

📐 Size/leading in CSS variables:
  --sz-h0, --sz-h1, --sz-h2, etc.
  --line-height-body
```

**✅ Maps to Postext:**
- `bodyText.fontFamily`, `bodyText.fontSize`, `bodyText.lineHeight`
- `headings.fontFamily` and `levels[n].fontSize`
- `headingStyles[n].fontFamily` for custom heading styles
- Font loading: `buildDocumentWithFonts()` with font resolver

**✅ Already Matches:**
- All font families are standard (David Libre, Bellefair, Frank Ruhl Libre, Noto Serif)
- Fontsource carries them all
- pt/em units align with Postext

---

#### 4. **Content Background System**
```
รשום ב: decorations.js (getContentBgCSS) + index.html

🎨 Content Page Backgrounds:
  cbg-regular-mode: 'none' | 'color' | 'texture' | 'image'
    → Regular content pages
  cbg-h0Open-mode, cbg-h2Open-mode: same options
    → Opening pages for H0, H2 headings

  Color: input[type=color] hex value
  Texture: CBG_TEXTURE_SVG (data URI with damask pattern)
  Image:
    - file upload → converts to base64
    - opacity slider (0-100) → fades with white overlay
    - applies via CSS background-image + linear-gradient

⚙️ Functions:
  getContentBgCSS(roleKey) → returns CSS string
  applyContentBg(pageEl, roleKey) → applies inline style
```

**✅ Maps to Postext:**
- Design slot background: `DesignBoxElement` with `backgroundColor`
- Page-wide background: place box at bleed in `headingStyles` or global `header`
- Texture: generate SVG pattern or use CSS
- Image: `DesignImageElement` with `resourceId` + opacity via box overlay

**⚠️ Gaps:**
- Postext backgrounds are per-element/slot, not global page property
- Image opacity handled differently (separate overlay box)
- Texture patterns must be SVG resources

---

#### 5. **Header/Footer (Running Heads)**
```
רשום ב: typesetter.js (createPageLayout)

📖 Running Head Configuration:
  Custom left/center/right content per page role
  Grid layout: 33.33% | 33.34% | 33.33% widths

  Functions:
  createPageLayout(..., customR, customC, customL) → sets running head
  resolveHeaderAlign(align, slot) → L/C/R positioning
  renderHeaderSectionHTML(type, ...) → generates content

  Types supported:
  - 'h1' → H1 title
  - 'h2', 'h2_with_siman', 'h2_without_siman'
  - 'h3', 'h4'
  - 'page_number' → folio
  - 'book_title', 'book_title_split_r', 'book_title_split_l'
  - 'first_word_last_word'
  - 'custom' → arbitrary text
  - 'custom_img' → image from data-hdr-c-img
  - Ornament types from HEADER_ORNAMENTS

🔧 Page Role Support:
  pageRole: 'h0Open' | 'h0' | 'h1' | 'h2Open' | 'h2' | 'h3' | 'regular'
  Each role may have different running head configuration
```

**✅ Maps to Postext:**
- `header.elements[]` array with text/rule/box/image elements
- Placement anchors: `top-left`, `top-center`, `top-right`
- Different headers per page parity: `parity: 'odd'` / `'even'`
- Page role filters: `pages: "body"` for non-opening pages
- Placeholders: `{chapterNumber}`, `{pageNumber}`, `{partTitle}`, etc.

**✅ Already Aligns:**
- Grid layout (33/33/33) standard in book design
- Page numbering, chapter titles already in Postext

---

#### 6. **Text Block Types & Formatting**
```
רשום ב: typesetter.js (applyDibburHamatchil, createDropWord, etc.)

📝 Drop Word / Capitular:
  drop-word-mode: 'window' | 'raised' | 'none'
  drop-word-size: number (lines)
  drop-word-font-size: multiplier
  Applied to first paragraph after heading

📝 Dibur Hamatchil (Talmud-specific):
  applyDibburHamatchil(text) → marks with ד"ה
  Talmud convention: first phrase in bold/special styling

📝 Indent After Heading:
  indentAfterHeading: false → no first-line indent after H

📝 Typography Details:
  optimalLineBreaking: true → Knuth-Plass paragraph breaking
  hyphenation: true
  widows/orphans control
  Rag optimization

⚙️ Methods:
  createDropWord(text, isContinuation, isSingleLine, numLines)
  getLinesFromParagraph(text, colWidth, styleObj, useWindowIfMultipleLines)
  applyDibburHamatchil(text)
```

**✅ Maps to Postext:**
- Drop caps: `bodyText.dropCap.kind` or `paragraphStyles[n].dropCap`
- Line breaking: `bodyText.optimalLineBreaking: true`
- Hyphenation: `bodyText.hyphenation: true`
- First-line indent: `bodyText.firstLineIndent`
- Widow/orphan control: `layout.orphanControl`, `widowControl`

**⚠️ Gaps:**
- Dibur Hamatchil (ד"ה) is Talmud-specific, not in Postext
  → Must use paragraph style + special markup (`:bold[ד"ה ...]`)
- Drop word "window" vs "raised" → Postext has `dropCap.kind: 'sunken' | 'raised'`

---

#### 7. **Grid & Layout Rules**
```
รשום ב: typesetter.js (updateGrid, balanceColumns, justifyColumnVertically)

📐 Column Layout:
  layoutType: 'double' (columns.json default)
  columnCount: configurable
  gutterWidth: 2% (configurable)
  columnRule: yes/no + style

📊 Column Balancing:
  balanceColumns(block, colWidth, styleObj, snapshot, availableHeight)
  Closes gaps with:
    - Grid lines above headings
    - Space after lists
    - Room under top floats
    - Loosened/tightened letter-spacing

📌 Baseline Grid:
  Not explicitly shown but implied in line-height calculations
  (baselineGridConfig in Postext)

🔲 Page Margins & Breaks:
  Mirrored margins: mirror: true
  Top/bottom/left/right margins in mm
  Page break parity control
```

**✅ Maps to Postext:**
- `layout.layoutType: 'double'`
- `layout.columnCount` / `gutterWidth`
- `layout.columnRule`
- `layout.columnBalancing`
- `page.margins.mirror: true`
- `headings.levels[n].breakBefore.parity`

**✅ Already Matches Postext's defaults**

---

#### 8. **Table of Contents & Front Matter**
```
รשום ב: typesetter.js (renderPaginatedTOC, buildStructuredTOC)

📑 TOC Generation:
  buildStructuredTOC(tokenList, showLevels, groupLevels)
  renderPaginatedTOC(container, entries, titleText, bookTitle, ...)

  Features:
  - Level filtering (show H1/H2/H3 etc)
  - Group by level
  - Page numbers attached to entries
  - Leaders (dots) between title and page number
```

**✅ Maps to Postext:**
- `toc.levels[]` config
- `toc.pageNumber` styling
- `toc.leader.gap` for dot leader
- `toc.subtitle` for entry descriptions

---

### Postext-Only Features (Not in IMUD)

1. **Resource Management:**
   - Figures, tables, images with numbering
   - Cross-references
   - Float placement (side channel, top, bottom)

2. **Typography (Advanced):**
   - CJK/Arabic/Japanese support (you have RTL, can add)
   - Kashida justification
   - Ruby/Furigana
   - Emphasis marks

3. **Output Formats:**
   - PDF (postext-pdf)
   - EPUB (postext-epub)
   - Canvas rendering
   - 3D book viewer (postext-folio)

4. **Advanced Features:**
   - Footnotes/endnotes with placement rules
   - Citations & bibliography
   - Code highlighting
   - Math rendering (MathJax)
   - Comics/manga layout (postext-comics)

---

### IMUD-Specific Features (NOT in Postext Core)

1. **Shaar Themes & Visual System:**
   - Pre-built theme sets (builtin_1, builtin_2, builtin_3)
   - Theme composition from SVG backgrounds
   - Drag/resize box editor on canvas
   - Custom theme builder with file uploads

2. **Ornament Library:**
   - Predefined left/right, up/down ornament sets
   - Dimension control per ornament
   - Fit-to-text auto-sizing for heading ornaments
   - Side ornament left/right mirror flip

3. **Content Backgrounds:**
   - Page-role-based backgrounds (regular, h0, h2, etc)
   - Texture patterns (damask)
   - Per-page-role image backgrounds with opacity

4. **Talmud-Specific:**
   - Dibur Hamatchil (ד"ה) marking
   - Siman numbering integration
   - Heading scope control (siman + title vs. separate)

5. **Shear Box Editor:**
   - Live drag/resize handles
   - Visual feedback on coordinates
   - Save as default template

---

## 📊 Mapping Summary Table

| IMUD Feature | Postext Equivalent | Status | Notes |
|---|---|---|---|
| Main Shaar | `headingStyles["cover"]` + design slot | ✅ Maps | Use advancedDesign with positioned elements |
| Back Shaar | `headingStyles["back"]` | ✅ Maps | Custom heading style |
| Sub Shaar | H1 + `advancedDesign` | ✅ Maps | Opening pages with design slot |
| Ornaments LR | Design slot text/box elements | ⚠️ Manual | No direct ornament registry |
| Ornaments UD | Design slot elements (positioned) | ⚠️ Manual | Must encode in slot |
| Section Dividers | Design slot or `heading.levels[2].advancedDesign` | ⚠️ Manual | No divider type system |
| Header/Footer | `header.elements[]`, `footer.elements[]` | ✅ Maps | Full feature support |
| Font config | `bodyText`, `headings.levels[]` | ✅ Maps | Direct alignment |
| Drop word | `bodyText.dropCap` | ✅ Maps | Partial (no "window" mode) |
| Dibur Hamatchil | Paragraph style + markup | ⚠️ Workaround | Not built-in |
| Content BG | Design slot box + image | ⚠️ Manual | Must construct per layout |
| Column balancing | `layout.columnBalancing` | ✅ Maps | Full support |
| TOC | `toc` config | ✅ Maps | Full support |
| Note rules | `footnotes.separatorConfig` | ⚠️ Limited | Fewer styling options |
| Siman numbering | Custom resource type | ⚠️ Workaround | Use resource numbering |

---

## 🔧 Export Strategy

### Phase 1: Direct Mappings
1. Extract all font selections → `bodyText`, `headings.levels[]`, `headingStyles[n]`
2. Export margin/layout settings → `page.margins`, `layout` config
3. Export running head config → `header.elements[]`, `footer.elements[]`
4. Export spacing/typography → `bodyText.fontSize`, `lineHeight`, etc.

### Phase 2: Design Slot Construction
1. For each shaar → generate `headingStyles[n].advancedDesign.slot`
   - Position boxes/text/images according to stored bounds
   - Encode SVG ornaments as base64 image resources
2. For each ornament → create positioned box/text in design slot
3. For backgrounds → create base box + image overlay

### Phase 3: Advanced Features
1. Register custom fonts → `customFonts[]` array
2. Build resource array for shaar images
3. Add custom heading styles for special page roles

---

## 📝 Next Steps

1. **Create TypeScript types** for IMUD config structure
2. **Build migration functions:**
   - `exportShaars()` → heading styles with design slots
   - `exportOrnaments()` → embedded in design slots
   - `exportBackgrounds()` → design slot boxes
   - `exportFonts()` → font config section
   - `exportLayout()` → page, layout, typography
3. **Generate complete `.json` file** compatible with Postext
4. **Create "Torani Preset" package** for easy import

