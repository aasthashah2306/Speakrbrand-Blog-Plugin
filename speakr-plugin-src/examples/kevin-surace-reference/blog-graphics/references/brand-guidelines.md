# Kevin Surace Brand Guidelines Reference

**Adapted for Kevin Surace's Blog & Thought Leadership Brand**  
Last Updated: April 2026

This document contains the design system for Kevin Surace's blog graphics, presentations, and content visuals.

---

## Design Philosophy

**Core Principles:**
- **Bold + Warm:** Dark base with energizing orange accents
- **Generous White Space**: 32px+ padding creates breathing room
- **Clear Visual Hierarchy**: Size, weight, and color create scannable content
- **Consistent Spacing**: Modular scale (16px, 24px, 32px, 40px, 64px, 96px)
- **Authoritative + Approachable**: Thought leader quality, not corporate sterility

---

## Color System

### Primary Palette

```
DARK PRIMARY (Brand Foundation)
Hex: #171420
RGB: 23, 20, 32
Usage: Primary backgrounds, headers, hero sections, brand moments
```

```
ACCENT ORANGE (Energy & Innovation)
Hex: #EE7700
RGB: 238, 119, 0
Usage: CTAs, highlights, key data points, badges, emphasis elements
```

```
LIGHT/WHITE (Space & Clarity)
Hex: #FFFEFF
RGB: 255, 254, 255
Usage: Text on dark backgrounds, card backgrounds, negative space
```

### Secondary Palette

```
LIGHT GRAY (Cards & Sections)
Hex: #F7F7FF
RGB: 247, 247, 255
Usage: Card backgrounds, alternating sections
```

```
DARK GRAY (Secondary Text)
Hex: #222222
RGB: 34, 34, 34
Usage: Body text on light backgrounds
```

```
MID GRAY (Tertiary Text)
Hex: #434343
RGB: 67, 67, 67
Usage: Supporting text, metadata
```

```
WARM ORANGE LIGHT (Tint)
Hex: #FFF4E6
RGB: 255, 244, 230
Usage: Subtle orange tint backgrounds, callout boxes
```

### Color Usage Guidelines

**For Headers:**
```
H1 (Main Title): #FFFEFF on #171420 backgrounds, or #171420 on light
H2 (Subheadlines): #171420 or #EE7700 depending on context
H3 (Subsection): #171420
Preheading: #EE7700 or #434343
```

**For Body Text:**
```
Primary text on light: #222222 or #171420
Secondary text: #434343
On dark backgrounds: #FFFEFF
Accent text: #EE7700
```

**For Data Visualization:**
```
Primary data: #EE7700 (Accent Orange)
Secondary data: #171420 (Dark Primary)
Tertiary data: #6B5B95 (Purple accent — complements orange)
Neutral data: #C8CCF2 (Supporting)
```

**For Backgrounds:**
```
Main background: #FFFEFF (White)
Section background: #171420 (Dark) or #F7F7FF (Light gray)
Card/container background: #FFFEFF or #F7F7FF
Hero: #171420 with gradient overlay
```

**For Interactive Elements:**
```
Buttons/CTAs: #EE7700 (Orange)
Hover states: #CC6600 (Darker orange)
Links: #EE7700 on light, #FFFEFF on dark
```

---

## Typography System

### Font Families

**Primary (Headings & Emphasis):**
- Font: Montserrat
- Weights: Bold (700), Medium (500), Medium Italic (500 italic)
- Usage: All headings, titles, emphasized content

**Secondary (Body & UI):**
- Font: Lato
- Weights: Bold (700), Regular (400)
- Usage: Body text, descriptions, UI elements

### Type Scale

#### Display/Hero Headings
```
H1 - Hero Title
Font: Montserrat Bold
Size: 56px
Line Height: 70px
Color: #FFFEFF on dark backgrounds, #171420 on light
Usage: Hero sections, major page titles
```

#### Section Headings
```
H2 - Section Title
Font: Montserrat Bold  
Size: 28px
Line Height: 35px
Color: #171420 (primary), #EE7700 (accent)
Usage: Card titles, section headings
```

```
H3 - Subsection Title
Font: Montserrat Bold
Size: 24px
Line Height: 29px
Color: #171420
Usage: Subsections, card categories
```

#### Body Text
```
Large Body
Font: Lato Regular
Size: 18px
Line Height: 29.25px
Color: #222222
Usage: Primary body content, list items
```

```
Small Body
Font: Lato Regular
Size: 16px
Line Height: 24px
Color: #222222, #999999 (metadata)
Usage: Captions, metadata, fine print
```

#### Special Elements
```
Badge Text
Font: Lato Bold
Size: 11px
Line Height: 16.5px
Letter Spacing: 0.5px
Transform: UPPERCASE
Color: White on #EE7700 or #171420
Usage: Status badges, category labels
```

---

## Spacing & Layout System

### Modular Spacing Scale

```
xs:   8px   - Tight internal spacing
sm:   16px  - Component internal padding
md:   24px  - Icon-to-content gaps, small sections
lg:   32px  - Card padding, section gaps
xl:   40px  - Major section gaps
2xl:  64px  - Large section margins
3xl:  96px  - Hero top margins
4xl:  120px - Maximum safe margins
```

### Container Patterns

#### Full-Width Container (Blog Infographic)
```
Width: 1200px
Side Margins: 64px (content width: 1072px)
Vertical Margins: 96px top, 96px bottom
```

#### Card Container
```
Width: 900px (infographic), 372px (services)
Padding: 32px all sides
Gap between cards: 32px vertical
```

### Major Section Spacing
```
Between major sections: 40px minimum
Between content and header: 32px
Top/bottom margins: 60px for blog graphics
```

---

## Component Patterns for Kevin's Content

### Topic Framework Infographic (Replaces Maturity Model)

**Overall Structure:**
- **Dimensions:** 1200px × variable (vertical blog format)
- **Background:** `#171420` with decorative gradients
- **Container:** 1072px content width (64px side margins)

#### Header Section
```yaml
Title:
  Font: Montserrat Bold 56px
  Color: #FFFEFF
  
Subtitle:
  Font: Montserrat Medium 24px
  Color: rgba(255, 254, 255, 0.9)
  Spacing: 16px gap from title
```

#### Content Cards
```yaml
Card:
  Width: 900px
  Background: #FFFEFF or #F7F7FF
  Padding: 32px
  
Icon Container:
  Background: #EE7700
  Size: 72px × 72px
  Icon: 40px white icon
  
Badge Colors (topic-based):
  AI Leadership: #EE7700
  Innovation: #6B5B95 (purple)
  Joy-Success: #10B981 (emerald)
  Curiosity: #2DE4E6 (cyan)
  Enterprise AI: #171420
```

### Hero Section Pattern
```
Background: #171420 with gradient
Title: Montserrat Bold 56-76px, #FFFEFF
Subtitle: Montserrat Medium 24px, rgba(255,254,255,0.9)
CTA Button: #EE7700, Lato Bold 16px, White text
```

---

## Quick Reference Cheat Sheet

```
PRIMARY DARK: #171420
ACCENT ORANGE: #EE7700
LIGHT/WHITE: #FFFEFF
BACKGROUND ALT: #F7F7FF

HEADER FONT: Montserrat Bold
BODY FONT: Lato Regular/Medium

H1: 56-76px, #FFFEFF or #171420
H2: 28px, #171420 or #EE7700
H3: 24px, #171420
Body: 18-20px, #222222

SPACING: 40px major sections, 20px within sections
MARGINS: 60-96px (blog), 40px (social), 80-120px (presentation)
CARD PADDING: 32px
ICON SIZES: 20px, 40px, 56px, 72px
```

---

## Brand Mistakes to Avoid

❌ **DON'T:**
- Use orange (#EE7700) for body text — only for accents, CTAs, highlights
- Mix Montserrat and Lato in same text block
- Use more than ONE H1 per graphic
- Use colors outside the defined palette
- Set body text below 16px
- Use MAN Digital blue (#000FC4) — this is Kevin Surace's brand

✅ **DO:**
- Use #171420 for primary dark elements
- Use #EE7700 for energy and call-to-action
- Apply Montserrat Bold only for headers
- Maintain 32-40px spacing between major sections
- Use lucide-react icons with proper specifications
- Test readability at 50% zoom

---

## Accessibility Requirements

### Color Contrast
- Text on white: Minimum 4.5:1 ratio (AA standard)
- Large text: Minimum 3:1 ratio
- #171420 on #FFFEFF: Passes AA ✅
- #EE7700 on #171420: Test carefully for small text (use for large/bold only)
- #FFFEFF on #171420: Passes AA ✅

---

**Document End**

This brand guidelines reference is adapted for Kevin Surace's thought leadership brand. Brand colors: #171420 (dark), #EE7700 (orange), #FFFEFF (white).
