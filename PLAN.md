# Development Plan: "Marginalia" Hugo Blog

## Context

Create a Hugo static blog called **"Marginalia"** (subtitle: "Notes from praying the Daily Office") with a custom theme evoking the typography of the **Episcopal Church's 1979 Book of Common Prayer** (the Sabon typeface, classical layout, black on white). The site deploys to GitHub Pages at `dailyoffice.prayerbooknotes.page` via GitHub Actions.

**Note:** All typographic references are to the Episcopal Church's 1979 BCP specifically — not the ACNA 2019 Book of Common Prayer, which is a different book with different typography.

---

## Prerequisites to Verify Before Starting

- [ ] Hugo is installed locally
- [ ] Sabon `.woff2` font files exist at `/Users/kevinbaker/source/artifacts`

---

## Phase 1: Hugo Project Initialization

1. **Run `hugo new site . --force`** in the existing repo (the `--force` flag allows a non-empty directory)
2. **Add hugo-PaperMod as a Git submodule:**
   ```
   git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
   ```
3. **Create `.gitignore`** (`public/`, `resources/`, `.hugo_build.lock`, `.DS_Store`)

### Why PaperMod (not Paper or Book)?

- **hugo-paper** now requires Tailwind CSS v4 + PostCSS (Node.js build step in CI) — unnecessary complexity
- **hugo-book** is documentation-oriented with sidebar navigation — wrong for a blog
- **hugo-PaperMod** uses plain CSS via Hugo Pipes (no Node.js), is blog-focused, natively supports tags/categories, and has a clean override path (`assets/css/extended/`)

### Theme Architecture: Overlay, Not Fork

PaperMod stays as a submodule. Customizations live in the project root via Hugo's lookup order:
- `assets/css/extended/prayerbook79.css` — all typography/layout overrides
- `static/fonts/` — Sabon woff2 files
- `layouts/partials/` — any template overrides (if needed)

This keeps the theme updatable and the customization layer clean.

---

## Phase 2: Configuration

Create **`hugo.toml`** with:
- `baseURL = "https://dailyoffice.prayerbooknotes.page/"`
- `theme = "PaperMod"`
- Light mode only (no dark mode toggle)
- `comments = false`
- Taxonomies: tags and categories
- Menu: About, Tags, Categories
- Home page info: title "Marginalia", subtitle "Notes from praying the Daily Office"

Create **`archetypes/default.md`** with front matter template (title, date, draft, tags, categories).

---

## Phase 3: PrayerBook79 Typography (the core of the project)

### 3.1: Copy Sabon fonts to `static/fonts/`

Four files: Sabon.woff2, SabonBold.woff2, SabonItalic.woff2, SabonBoldItalic.woff2

### 3.2: Create `assets/css/extended/prayerbook79.css`

Design principles drawn from examining the first 50 pages of the **1979 BCP** PDF:

#### Typographic Hierarchy (observed from the BCP)

The BCP uses a **size-driven hierarchy at the top levels**, reserving bold for mid-level headings only:

| Level | BCP Usage | Weight | Kerning | Blog Equivalent |
|---|---|---|---|---|
| Display | Title page: "The Book of Common Prayer" | Roman (400) | Very tight | Site title |
| Section title | "Preface", "Daily Morning Prayer: Rite One" | Roman (400) | Tight | Post title |
| Content heading | "Confession of Sin", "The Invitatory and Psalter" | **Bold (700)** | Normal | H2 in posts |
| Sub-heading | "Jubilate", numbered sections like "1. Principal Feasts" | **Bold (700)** | Normal | H3 in posts |
| Body text | Preface prose, rubrical explanations | Roman (400) | Normal | Body paragraphs |
| Rubrics | "The Officiant says to the people", "Silence may be kept." | *Italic (400)* | Normal | Blockquotes, emphasis |
| Scripture refs | *Isaiah 40:3*, *Psalm 100* | *Italic (400)*, smaller | Normal | Inline citations |
| Running footer | Page number + italic section name | Roman + *Italic* | Normal | Footer |

#### Key Typographic Details

- **Alignment:** Body text is left-aligned with a ragged right edge (not justified)
- **Paragraph spacing:** Paragraphs separated by vertical space, not first-line indentation
- **Large headings are roman, not bold** — size and tight kerning alone create the visual weight. This is critical to the BCP aesthetic.
- **Bold appears only at mid-level** — content sub-headings like "Confession of Sin" and numbered sections like "1. Principal Feasts" use bold
- **Rubrics are always italic** — every liturgical instruction, stage direction, and contextual note
- **Running footers:** Page number on outer edge, italic section name adjacent
- **Generous whitespace:** Large vertical gaps between section heading and first paragraph; sections "breathe"
- **Verse/poetry:** Hanging indentation for psalms and canticles (continuation lines indented)

#### Design Mapping to CSS

| Principle | 1979 BCP | Web Implementation |
|---|---|---|
| Typeface | Sabon (Jan Tschichold, 1964-67) | `@font-face` with woff2, Georgia fallback |
| Color | Black ink on white paper | `#1a1a1a` on `#ffffff` |
| Line length | ~60-70 characters | `max-width: 680px` |
| Leading | Generous for sustained reading | Fluid `line-height` via `clamp()` |
| Alignment | Left-aligned, ragged right | `text-align: left` (no justification) |
| Top-level headings | Very large roman with tight kerning | `font-weight: 400`, negative `letter-spacing` |
| Mid-level headings | Bold at a moderate size | `font-weight: 700`, normal kerning |
| Rubrics/instructions | Italic throughout | `font-style: italic` on blockquotes |
| Paragraph separation | Vertical space, no indent | `margin-bottom` on `<p>`, no `text-indent` |
| Whitespace | Generous margins above headings | Large `margin-top` on headings |
| Responsive | N/A (print) | Fluid typography with `clamp()`, mobile-first |

### Responsive Design with `clamp()`

All font sizes and key spacing values will use CSS `clamp()` for smooth scaling between mobile and desktop — no abrupt breakpoint jumps:

- **Body text:** `clamp(16px, 1.1vw + 14px, 19px)` — 16px on small screens, scales to 19px on desktop
- **Post titles:** `clamp(28px, 4vw + 12px, 38px)` with `font-weight: 400` and `letter-spacing: -0.02em` — large roman, tightly kerned, as in BCP section title pages
- **H2 (content headings):** `clamp(22px, 2.5vw + 10px, 28px)` with `font-weight: 700` and `letter-spacing: -0.01em` — bold, as in BCP content headings like "Confession of Sin"
- **H3 (sub-headings):** `clamp(18px, 1.5vw + 10px, 20px)` with `font-weight: 700` — bold, as in BCP numbered sections
- **H1 (rare, within post body):** `clamp(26px, 3.5vw + 10px, 34px)` with `font-weight: 400` and `letter-spacing: -0.015em` — roman, matching BCP section title style
- **Line height:** `clamp(1.6, 1.5 + 0.2vw, 1.7)` — slightly tighter on mobile for better use of space
- **Content max-width:** remains `680px` but with fluid horizontal padding so content never hits screen edges

### Additional CSS Details

- **Blockquotes** styled as BCP rubrics: italic, no heavy border, subtle left margin
- **Running footer** in site footer: small text, italic section context, matching BCP footer style
- **Horizontal rules** (`<hr>`): thin, understated, centered at partial width — used sparingly as in the BCP
- **Links:** underlined with a warm gray (`#8b7d6b`) underline color, evoking aged paper; darken on hover
- **Suppress dark mode entirely** — the BCP is definitively black-on-white

The CSS will:
- Declare `@font-face` for all four Sabon weights/styles
- Override PaperMod's CSS custom properties (`:root` vars) for colors
- Use `clamp()` throughout for fluid, responsive typography and spacing
- Implement the BCP's size-driven hierarchy (roman for large headings, bold for mid-level)
- Style blockquotes as italic rubrics
- Use warm gray accents for borders/underlines
- Suppress dark mode entirely

---

## Phase 4: Content

1. **`content/about.md`** — About page (linked from nav menu)
2. **`content/posts/first-post.md`** — Starter post to verify the build (can be placeholder or real)

---

## Phase 5: GitHub Actions & Deployment

### 5.1: Create `.github/workflows/hugo.yaml`

- Trigger: push to `main` + manual dispatch
- Install Hugo Extended (no Node.js needed)
- Checkout with `submodules: recursive` and `fetch-depth: 0`
- Build with `hugo --gc --minify`
- Deploy via `actions/deploy-pages@v4`

### 5.2: Create `static/CNAME`

Contains `dailyoffice.prayerbooknotes.page` — persists across deployments.

### 5.3: Manual steps (user must do)

- Configure GitHub repo Settings > Pages > Source: **GitHub Actions**
- Set custom domain to `dailyoffice.prayerbooknotes.page`
- Add DNS CNAME record: `dailyoffice.prayerbooknotes.page → kevin-w-baker.github.io`

---

## Files Created (Complete List)

```
.github/workflows/hugo.yaml          # CI/CD
.gitignore
archetypes/default.md                 # Post template
assets/css/extended/prayerbook79.css  # Custom typography
content/about.md                      # About page
content/posts/first-post.md          # Starter post
static/CNAME                          # Custom domain
static/fonts/Sabon.woff2             # Copied from ../../artifacts/
static/fonts/SabonBold.woff2
static/fonts/SabonItalic.woff2
static/fonts/SabonBoldItalic.woff2
themes/PaperMod/                      # Git submodule (not manually created)
hugo.toml                             # Site configuration
```

---

## Verification

1. Run `hugo server` locally — confirm site loads with Sabon typography, correct layout, no dark mode
2. Verify responsive scaling — resize browser from mobile to desktop widths, confirm `clamp()` typography scales smoothly
3. Check tag/category pages work from the starter post
4. Check About page renders and is linked from nav
5. Commit and push — confirm GitHub Actions build succeeds
6. After DNS propagation, verify `https://dailyoffice.prayerbooknotes.page/` serves the site
