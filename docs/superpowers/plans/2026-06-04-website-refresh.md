# Website Refresh Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restyle the al-folio personal site (editorial serif look, dark-default, teal accent from the lab logo) and add data-driven Lab and Teaching pages, while deleting stale template pages.

**Architecture:** Restyle *within* al-folio — retarget its CSS custom properties in `_sass/_themes.scss`, add one new SCSS partial `_sass/_refresh.scss` for typography + new components, flip the theme-init default in `assets/js/theme.js`, and add two Jekyll data files (`_data/lab.yml`, `_data/courses.yml`) rendered by two new includes. No engine/plugin changes.

**Tech Stack:** Jekyll, Liquid, SCSS (compiled by `jekyll-sass-converter`), Bootstrap 4.6 (already loaded), jekyll-scholar. Ruby/Bundler toolchain.

**Spec:** `docs/superpowers/specs/2026-06-04-website-refresh-design.md`

**Branch:** `website-refresh` (already created; the spec is committed there).

> **Testing note — read first.** This is a static Jekyll site with **no unit-test framework**. The "test" for each task is: (1) a clean production build — `bundle exec jekyll build` must exit 0; (2) a targeted assertion against the generated `_site/` output (a `grep` with expected result) where applicable; (3) a final visual pass via `bundle exec jekyll serve`. Each task ends with a commit. If native gems fail to build locally, use `docker compose up` instead of the bundler commands.

---

## File map

**Create**
- `_sass/_refresh.scss` — new typography + nav + reading-width + lab + teaching component styles.
- `_data/lab.yml` — lab member data (PI / collaborators / current / past).
- `_data/courses.yml` — course data with per-term offerings + syllabus slugs.
- `_includes/lab_members.html` — renders lab groups as panels of member cards.
- `_includes/courses.html` — renders course cards.
- `_pages/lab.md` — `/lab/` page.
- `assets/img/favicon.svg` — square network-style favicon derived from the lab logo.

**Modify**
- `_includes/head.html` — Google Fonts links (Newsreader + Inter; keep Material Icons).
- `_sass/_base.scss` — remove forced Noto Sans + `font-weight:500!important`.
- `_sass/_themes.scss` — retarget `--global-*` custom properties + add `--accent-soft`, `--avatar-bg`.
- `assets/css/main.scss` — add `"refresh"` to the `@import` list.
- `assets/js/theme.js` — default to dark when no stored choice.
- `_config.yml` — `icon: favicon.svg`.
- `_pages/teaching.md` — replace template body with intro + courses include.
- `_pages/publications.md` — (only if needed) keep `nav_order: 1`.
- `_pages/about.md` — add "Join the lab" CTA.

**Delete**
- `_pages/projects.md`, `_pages/repositories.md`, `_pages/collaborators.md`, `_data/repositories.yml`, `assets/pdf/syllabi/.DS_Store`.

**Rename (web-safe syllabus slugs)** — see Task 8.

---

## Task 0: Environment + baseline

**Files:** none (setup only)

- [ ] **Step 1: Confirm branch**

Run: `git branch --show-current`
Expected: `website-refresh`

- [ ] **Step 2: Install gems**

Run: `bundle install`
Expected: completes without error (creates `Gemfile.lock`, gitignored). If native gems (mini_racer, jekyll-imagemagick) fail, stop and use Docker for all build/serve steps (`docker compose up`).

- [ ] **Step 3: Baseline build**

Run: `bundle exec jekyll build`
Expected: exit 0, `_site/` generated. This is the known-good baseline before changes.

- [ ] **Step 4: (Optional) commit the /init CLAUDE.md**

`CLAUDE.md` was generated during setup and is untracked. Commit it so it isn't orphaned:
```bash
git add CLAUDE.md && git commit -m "docs: add CLAUDE.md from project init"
```

---

## Task 1: Fonts (Newsreader + Inter)

**Files:**
- Modify: `_includes/head.html:25`
- Modify: `_sass/_base.scss:7-26`

- [ ] **Step 1: Swap the Google Fonts links in `_includes/head.html`**

Find line 25:
```html
    <link rel="stylesheet" type="text/css" href="https://fonts.googleapis.com/css?family=Noto+Sans:300,400,500,700|Material+Icons">
```
Replace with (two tags — keep Material Icons, add Newsreader + Inter):
```html
    <link rel="stylesheet" type="text/css" href="https://fonts.googleapis.com/css?family=Material+Icons">
    <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Newsreader:opsz,wght@6..72,400;6..72,500;6..72,600;6..72,700&display=swap">
```

- [ ] **Step 2: Remove the forced Noto Sans + weight in `_sass/_base.scss`**

Find lines 7-26:
```scss
html {
  font-weight: 500!important;
}

p,
h1,
h2,
h3,
h4,
h5,
h6,
em,
div,
li,
span,
strong {
  color: var(--global-text-color);
  font-family: 'Noto Sans';
  font-weight: 500!important;
}
```
Replace with (keep the text color binding, drop the font-family + !important — `_refresh.scss` sets fonts):
```scss
p,
h1,
h2,
h3,
h4,
h5,
h6,
em,
div,
li,
span,
strong {
  color: var(--global-text-color);
}
```

- [ ] **Step 3: Build**

Run: `bundle exec jekyll build`
Expected: exit 0.

- [ ] **Step 4: Assert the new font link is in output**

Run: `grep -c "Newsreader" _site/index.html`
Expected: `1` (or greater).

- [ ] **Step 5: Commit**

```bash
git add _includes/head.html _sass/_base.scss
git commit -m "style: load Newsreader + Inter, drop forced Noto Sans"
```

---

## Task 2: Palette tokens (themes.scss)

**Files:**
- Modify: `_sass/_themes.scss` (the `:root` block and the `html[data-theme='dark']` block)

Set al-folio's existing `--global-*` custom properties to the refresh palette, and add two new properties (`--accent-soft`, `--avatar-bg`). Leave the tip/warning/danger block variables and the `.fa-sun`/`.fa-moon`/`.repo-img-*` rules unchanged.

- [ ] **Step 1: Update the `:root` (light) block**

In `_sass/_themes.scss`, within `:root { ... }`, replace these property lines with:
```scss
  --global-bg-color: #f6f7f5;
  --global-code-bg-color: #ffffff;
  --global-text-color: #21302d;
  --global-text-color-light: #5f6b66;
  --global-text-color-lighter: #8a938e;
  --global-theme-color: #1d7a72;
  --global-theme-color-light: rgba(29,122,114,.14);
  --global-theme-color-lightest: rgba(29,122,114,.07);
  --global-hover-color: #155f59;
  --global-hover-text-color: #ffffff;
  --global-footer-bg-color: #eef0ed;
  --global-footer-text-color: #5f6b66;
  --global-footer-link-color: #1d7a72;
  --global-distill-app-color: #5f6b66;
  --global-divider-color: #e2e5e1;
  --global-card-bg-color: #ffffff;
  --accent-soft: rgba(29,122,114,.14);
  --avatar-bg: #cfd6d2;
```

- [ ] **Step 2: Update the `html[data-theme='dark']` block**

Within `html[data-theme='dark'] { ... }`, replace these property lines with:
```scss
  --global-bg-color: #1a201f;
  --global-code-bg-color: #222a28;
  --global-text-color: #e7e6e1;
  --global-text-color-light: #92978f;
  --global-text-color-lighter: #6f746d;
  --global-theme-color: #27a89e;
  --global-theme-color-light: rgba(39,168,158,.16);
  --global-theme-color-lightest: rgba(39,168,158,.08);
  --global-hover-color: #34c0b4;
  --global-hover-text-color: #11100f;
  --global-footer-bg-color: #151918;
  --global-footer-text-color: #92978f;
  --global-footer-link-color: #27a89e;
  --global-distill-app-color: #e7e6e1;
  --global-divider-color: #33403d;
  --global-card-bg-color: #222a28;
  --accent-soft: rgba(39,168,158,.16);
  --avatar-bg: #46514f;
```

- [ ] **Step 3: Build**

Run: `bundle exec jekyll build`
Expected: exit 0.

- [ ] **Step 4: Assert compiled CSS contains the dark bg**

Run: `grep -ro "#1a201f" _site/assets/css/ | head -1`
Expected: a match (the new dark bg compiled into `main.css`).

- [ ] **Step 5: Commit**

```bash
git add _sass/_themes.scss
git commit -m "style: retarget theme palette to teal/charcoal refresh"
```

---

## Task 3: Dark mode as default

**Files:**
- Modify: `assets/js/theme.js:90-99` (`initTheme`)

- [ ] **Step 1: Default to dark when no stored choice**

Find:
```js
let initTheme = (theme) => {
  if (theme == null || theme == "null") {
    const userPref = window.matchMedia;
    if (userPref && userPref("(prefers-color-scheme: dark)").matches) {
      theme = "dark";
    }
  }

  setTheme(theme);
};
```
Replace with:
```js
let initTheme = (theme) => {
  // Default to dark when the visitor has no saved preference.
  if (theme == null || theme == "null") {
    theme = "dark";
  }

  setTheme(theme);
};
```

- [ ] **Step 2: Build**

Run: `bundle exec jekyll build`
Expected: exit 0.

- [ ] **Step 3: Assert the default landed in output**

Run: `grep -c 'theme = "dark";' _site/assets/js/theme.js`
Expected: `1`. (If the minifier rewrote it, instead run `grep -c "dark" _site/assets/js/theme.js` and expect ≥ 1.)

- [ ] **Step 4: Commit**

```bash
git add assets/js/theme.js
git commit -m "feat: default to dark mode when no saved preference"
```

---

## Task 4: Refresh stylesheet partial

**Files:**
- Create: `_sass/_refresh.scss`
- Modify: `assets/css/main.scss` (add `"refresh"` to `@import`)

- [ ] **Step 1: Create `_sass/_refresh.scss`**

```scss
/*******************************************************************************
 * Website refresh — typography, nav, reading width, and Lab/Teaching
 * components. Layered on al-folio base styles. Colors come from the
 * --global-* / --accent-* custom properties in _themes.scss.
 ******************************************************************************/

// --- Typography ---------------------------------------------------------
body, p, em, li, span, strong, a, td, th, input, button, .nav-link {
  font-family: 'Inter', sans-serif;
}

h1, h2, h3, h4, h5, h6,
.post-title,
.navbar-brand.title {
  font-family: 'Newsreader', serif;
  font-weight: 600;
  letter-spacing: -0.01em;
}

// --- Reading measure ----------------------------------------------------
.post > article > p,
.post > article > ul,
.post > article > ol,
.post > article > blockquote,
.post-description {
  max-width: 760px;
}

// --- Nav active state ---------------------------------------------------
.navbar .nav-item.active .nav-link {
  color: var(--global-theme-color);
  border-bottom: 2px solid var(--global-theme-color);
}

// --- Accent button ------------------------------------------------------
.btn-accent {
  display: inline-block;
  background: var(--global-theme-color);
  color: var(--global-hover-text-color);
  font-weight: 600;
  font-size: 0.85rem;
  padding: 0.55rem 1.1rem;
  border-radius: 8px;
  text-decoration: none;
  &:hover {
    background: var(--global-hover-color);
    color: var(--global-hover-text-color);
    text-decoration: none;
  }
}

// --- Lab page -----------------------------------------------------------
.lab-logo {
  text-align: center;
  margin: 0 auto 2rem;
  img { max-width: 360px; width: 70%; height: auto; }
}
.lab-group { margin-bottom: 2.5rem; }
.lab-group-title {
  font-size: 1.4rem;
  border-bottom: 1px solid var(--global-divider-color);
  padding-bottom: 0.4rem;
  margin-bottom: 1.3rem;
}
.lab-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 1.1rem;
}
.lab-card {
  background: var(--global-card-bg-color);
  border: 1px solid var(--global-divider-color);
  border-radius: 12px;
  padding: 1.2rem;
  text-align: center;
}
.lab-card--pi { border-color: var(--global-theme-color); }
.lab-avatar {
  width: 84px; height: 84px;
  margin: 0 auto 0.8rem;
  border-radius: 50%;
  overflow: hidden;
  background: var(--avatar-bg);
  display: flex; align-items: center; justify-content: center;
  img { width: 100%; height: 100%; object-fit: cover; }
}
.lab-initials { font-weight: 600; font-size: 1.5rem; color: #fff; }
.lab-name { font-weight: 600; font-size: 1rem; }
.lab-role { color: var(--global-text-color-light); font-size: 0.85rem; margin-top: 0.1rem; }
.lab-bio { font-size: 0.85rem; margin-top: 0.5rem; color: var(--global-text-color); }
.lab-badge {
  display: inline-block;
  margin-top: 0.6rem;
  font-size: 0.65rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  font-weight: 600;
  padding: 0.18rem 0.6rem;
  border-radius: 20px;
}
.lab-badge--pi { background: var(--global-theme-color); color: var(--global-hover-text-color); }
.lab-badge--current { background: var(--accent-soft); color: var(--global-theme-color); }
.lab-badge--alum { background: var(--global-divider-color); color: var(--global-text-color-light); }
.lab-links {
  margin-top: 0.6rem;
  a {
    color: var(--global-text-color-light);
    margin: 0 0.3rem;
    font-size: 1rem;
    &:hover { color: var(--global-theme-color); }
  }
}

// --- Teaching page ------------------------------------------------------
.course-list { display: flex; flex-direction: column; gap: 1.3rem; }
.course-card {
  background: var(--global-card-bg-color);
  border: 1px solid var(--global-divider-color);
  border-radius: 12px;
  padding: 1.3rem 1.5rem;
}
.course-head { display: flex; align-items: baseline; gap: 0.8rem; flex-wrap: wrap; }
.course-code {
  font-weight: 700;
  color: var(--global-theme-color);
  font-size: 0.9rem;
  letter-spacing: 0.02em;
}
.course-level {
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--global-text-color-light);
  border: 1px solid var(--global-divider-color);
  border-radius: 20px;
  padding: 0.1rem 0.6rem;
}
.course-title { font-size: 1.3rem; margin: 0.3rem 0 0.5rem; }
.course-summary { font-size: 0.92rem; color: var(--global-text-color); max-width: 70ch; }
.course-topics { display: flex; flex-wrap: wrap; gap: 0.4rem; margin: 0.6rem 0; }
.topic-chip {
  font-size: 0.72rem;
  background: var(--accent-soft);
  color: var(--global-theme-color);
  padding: 0.15rem 0.6rem;
  border-radius: 6px;
}
.course-terms { display: flex; flex-wrap: wrap; gap: 0.5rem; margin-top: 0.8rem; }
.term-btn {
  font-size: 0.78rem;
  border: 1px solid var(--global-divider-color);
  border-radius: 7px;
  padding: 0.35rem 0.7rem;
  color: var(--global-text-color);
  text-decoration: none;
  i { color: var(--global-theme-color); margin-left: 0.3rem; }
  &:hover { border-color: var(--global-theme-color); text-decoration: none; }
}
.term-btn--nolink { opacity: 0.6; }
.course-note { font-size: 0.8rem; color: var(--global-text-color-light); margin-top: 0.6rem; }
```

- [ ] **Step 2: Import the partial in `assets/css/main.scss`**

Find the `@import` list and add `"refresh",` right after `"base",`:
```scss
@import
  "variables",
  "themes",
  "layout",
  "base",
  "refresh",
  "distill",
  "cv",
  "font-awesome/fontawesome",
  "font-awesome/brands",
  "font-awesome/solid",
  "font-awesome/regular"
;
```

- [ ] **Step 3: Build**

Run: `bundle exec jekyll build`
Expected: exit 0 (SCSS compiles; a syntax error here fails the build).

- [ ] **Step 4: Assert refresh styles compiled**

Run: `grep -ro "lab-card" _site/assets/css/ | head -1`
Expected: a match.

- [ ] **Step 5: Commit**

```bash
git add _sass/_refresh.scss assets/css/main.scss
git commit -m "style: add refresh partial (fonts, nav, lab + teaching components)"
```

---

## Task 5: Navigation + delete stale pages

**Files:**
- Delete: `_pages/projects.md`, `_pages/repositories.md`, `_pages/collaborators.md`, `_data/repositories.yml`

- [ ] **Step 1: Confirm nothing else references the data file**

Run: `grep -rn "repositories" _pages _includes _layouts _config.yml | grep -v "_pages/repositories.md"`
Expected: the only remaining hits are inside `_includes/repositories.html` / `repository.html` (the now-orphaned partials) and the about/news — **no other page consumes `_data/repositories.yml`**. If a live page (other than the ones being deleted) uses it, stop and report.

- [ ] **Step 2: Delete the files**

```bash
git rm _pages/projects.md _pages/repositories.md _pages/collaborators.md _data/repositories.yml
```

- [ ] **Step 3: Build**

Run: `bundle exec jekyll build`
Expected: exit 0 (no Liquid error from a dangling reference).

- [ ] **Step 4: Assert the pages are gone and nav is intact**

Run: `test ! -e _site/projects/index.html && test ! -e _site/repositories/index.html && test ! -e _site/collaborators/index.html && echo OK`
Expected: `OK`

Run: `grep -o 'href="/publications/"' _site/index.html | head -1`
Expected: a match (publications still in nav). (Lab + teaching nav verified in Tasks 6–7.)

- [ ] **Step 5: Commit**

```bash
git commit -m "chore: remove stale template pages (projects, repositories, collaborators)"
```

---

## Task 6: Lab page (data + include + page)

**Files:**
- Create: `_data/lab.yml`
- Create: `_includes/lab_members.html`
- Create: `_pages/lab.md`

- [ ] **Step 1: Create `_data/lab.yml`** (placeholders only — do not invent real people)

```yaml
pi:
  - name: Satyaki Sikdar
    role: Principal Investigator
    image: satyaki-2024-bw.png
    bio: >
      Assistant Professor of Computer Science at Loyola University Chicago,
      working at the confluence of network science, machine learning, and
      computational social science.
    links:
      website: https://satyaki.net
      scholar: https://scholar.google.com/citations?user=2f6t6hYAAAAJ
      github: https://github.com/satyakisikdar

collaborators:
  - name: Collaborator Name
    role: "Professor, Department · University"
    image:
    bio: >
      Placeholder — short description of the collaboration. Replace.
    links:
      website: ""

current:
  - name: Student Name
    role: PhD Student
    since: 2025
    image:
    bio: >
      Placeholder — research interests. Replace.
    links:
      github: ""

past:
  - name: Alum Name
    role: "MS '25"
    now: "Now at Company / School"
    image:
    links:
      website: ""
```

- [ ] **Step 2: Create `_includes/lab_members.html`**

```liquid
{%- assign groups = "pi,collaborators,current,past" | split: "," -%}
{%- assign titles = "Principal Investigator,Collaborators,Current Members,Alumni" | split: "," -%}

{%- for group in groups -%}
  {%- assign members = site.data.lab[group] -%}
  {%- if members and members.size > 0 -%}

  {%- case group -%}
    {%- when 'pi' -%}{%- assign gtitle = titles[0] -%}{%- assign blabel = 'PI' -%}{%- assign bclass = 'lab-badge--pi' -%}
    {%- when 'collaborators' -%}{%- assign gtitle = titles[1] -%}{%- assign blabel = 'Faculty' -%}{%- assign bclass = 'lab-badge--alum' -%}
    {%- when 'current' -%}{%- assign gtitle = titles[2] -%}{%- assign blabel = 'Current' -%}{%- assign bclass = 'lab-badge--current' -%}
    {%- when 'past' -%}{%- assign gtitle = titles[3] -%}{%- assign blabel = 'Alum' -%}{%- assign bclass = 'lab-badge--alum' -%}
  {%- endcase -%}

  <section class="lab-group">
    <h2 class="lab-group-title">{{ gtitle }}</h2>
    <div class="lab-grid">
      {%- for m in members -%}
      <div class="lab-card{% if group == 'pi' %} lab-card--pi{% endif %}">
        <div class="lab-avatar">
          {%- if m.image and m.image != "" -%}
          <img src="{{ m.image | prepend: '/assets/img/' | relative_url }}" alt="{{ m.name }}">
          {%- else -%}
          {%- assign parts = m.name | strip | split: " " -%}
          <span class="lab-initials">{{ parts.first | slice: 0, 1 | upcase }}{{ parts.last | slice: 0, 1 | upcase }}</span>
          {%- endif -%}
        </div>
        <div class="lab-name">{{ m.name }}</div>
        {%- if m.role and m.role != "" -%}<div class="lab-role">{{ m.role }}</div>{%- endif -%}
        {%- if m.since -%}<div class="lab-role">Since {{ m.since }}</div>{%- endif -%}
        {%- if m.now and m.now != "" -%}<div class="lab-role">{{ m.now }}</div>{%- endif -%}
        {%- if m.bio and m.bio != "" -%}<div class="lab-bio">{{ m.bio }}</div>{%- endif -%}
        <span class="lab-badge {{ bclass }}">{{ blabel }}</span>
        {%- if m.links -%}
        <div class="lab-links">
          {%- if m.links.website and m.links.website != "" -%}<a href="{{ m.links.website }}" target="_blank" rel="noopener" title="Website"><i class="fas fa-globe"></i></a>{%- endif -%}
          {%- if m.links.scholar and m.links.scholar != "" -%}<a href="{{ m.links.scholar }}" target="_blank" rel="noopener" title="Google Scholar"><i class="ai ai-google-scholar"></i></a>{%- endif -%}
          {%- if m.links.github and m.links.github != "" -%}<a href="{{ m.links.github }}" target="_blank" rel="noopener" title="GitHub"><i class="fab fa-github"></i></a>{%- endif -%}
        </div>
        {%- endif -%}
      </div>
      {%- endfor -%}
    </div>
  </section>
  {%- endif -%}
{%- endfor -%}
```

- [ ] **Step 3: Create `_pages/lab.md`**

```markdown
---
layout: page
title: lab
permalink: /lab/
nav: true
nav_order: 2
description: The Sikdar Lab — network science, machine learning, and computational social science.
---

<div class="lab-logo">
  <img class="repo-img-light" src="{{ '/assets/img/lab-logo-light.png' | relative_url }}" alt="Sikdar Lab">
  <img class="repo-img-dark" src="{{ '/assets/img/lab-logo-dark.png' | relative_url }}" alt="Sikdar Lab">
</div>

The **Sikdar Lab** studies the fundamental mechanisms that govern complex,
interconnected systems, using tools at the confluence of *network science*,
*machine learning*, and *computational social science*.

**Prospective students:** I am looking for motivated students interested in
these areas. If that's you, please reach out — see the
[contact details](/) on the about page.

{% include lab_members.html %}
```

- [ ] **Step 4: Build**

Run: `bundle exec jekyll build`
Expected: exit 0 (Liquid in the include compiles cleanly).

- [ ] **Step 5: Assert the lab page renders members + nav link**

Run: `grep -c "lab-card" _site/lab/index.html`
Expected: ≥ `4` (PI + one per placeholder group).

Run: `grep -o 'href="/lab/"' _site/index.html | head -1`
Expected: a match (Lab now in nav).

Run: `grep -o ">SS<" _site/lab/index.html | head -1`
Expected: a match (PI has an image, so this may be empty — instead verify a placeholder initials avatar): `grep -o 'lab-initials' _site/lab/index.html | head -1` → a match.

- [ ] **Step 6: Commit**

```bash
git add _data/lab.yml _includes/lab_members.html _pages/lab.md
git commit -m "feat: add data-driven Lab page with member panels"
```

---

## Task 7: Teaching page (rename syllabi + data + include + page)

**Files:**
- Rename: 8 PDFs in `assets/pdf/syllabi/`
- Delete: `assets/pdf/syllabi/.DS_Store`
- Create: `_data/courses.yml`
- Create: `_includes/courses.html`
- Modify: `_pages/teaching.md`

- [ ] **Step 1: Rename syllabi to web-safe slugs**

The files are untracked, so use plain `mv`:
```bash
cd assets/pdf/syllabi
mv "Fall 2024 - Comp 141 Combined.pdf"      comp141-intro-computing-tools-fall2024.pdf
mv "Fall 2024 - COMP 406.pdf"               comp406-data-mining-fall2024.pdf
mv "Spring 2025 - COMP 306-406.pdf"         comp306-406-data-mining-spring2025.pdf
mv "Fall 2025 - COMP 306-406.pdf"           comp306-406-data-mining-fall2025.pdf
mv "Spring 2026 - COMP 306-406.pdf"         comp306-406-data-mining-spring2026.pdf
mv "Fall 2025 - COMP271 SyllabusV6.pdf"     comp271-data-structures-fall2025.pdf
mv "Spring 2025 - COMP 388:488 Net Sci.pdf" comp388-488-network-science-spring2025.pdf
mv "Spring 2026 - HONR 204-03H.pdf"         honr204-science-society-network-science-spring2026.pdf
rm -f .DS_Store
cd ../../..
```

- [ ] **Step 2: Verify the renames**

Run: `ls assets/pdf/syllabi/`
Expected: 8 `.pdf` files, all lowercase-hyphenated, no spaces/colons, no `.DS_Store`.

- [ ] **Step 3: Create `_data/courses.yml`** (ordered most-recent first; summaries are editable drafts)

```yaml
- code: "COMP 306/406"
  title: "Data Mining"
  level: "Undergraduate + Graduate"
  summary: >
    Theory and practice of analyzing very large datasets: structured and
    unstructured data, ETL with Pandas, cleaning, visualization, frequent-
    pattern mining, clustering, and supervised learning. The 406 (graduate)
    section adds research components and formal writing.
  topics: [Pandas ETL, data cleaning, visualization, frequent-pattern mining, clustering, supervised learning]
  offerings:
    - { term: "Spring 2026", modality: "In person", syllabus: "comp306-406-data-mining-spring2026.pdf" }
    - { term: "Fall 2025",   modality: "In person", syllabus: "comp306-406-data-mining-fall2025.pdf" }
    - { term: "Spring 2025", modality: "In person", syllabus: "comp306-406-data-mining-spring2025.pdf" }
    - { term: "Fall 2024",   modality: "Online",    syllabus: "comp406-data-mining-fall2024.pdf" }
  note: ""

- code: "HONR 204"
  title: "Science & Society: Network Science"
  level: "Honors"
  summary: >
    Honors seminar: measuring network structure, identifying key nodes and
    communities, and modeling contagion — blending theory with real-world
    examples across computer science, sociology, political science, and biology.
  topics: [network structure, centrality, community detection, contagion, resilience]
  offerings:
    - { term: "Spring 2026", modality: "In person", syllabus: "honr204-science-society-network-science-spring2026.pdf" }
  note: ""

- code: "COMP 271"
  title: "Data Structures I"
  level: "Undergraduate (+ grad COMP 400B)"
  summary: >
    Data abstraction and structures — stacks, queues, lists, sets, and trees —
    with abstract data types, recursion, sorting and searching, and the analysis
    of correctness and efficiency. Programming-intensive (Python).
  topics: [stacks, queues, lists, sets, trees, ADTs, recursion, sorting & searching]
  offerings:
    - { term: "Fall 2025", modality: "In person", syllabus: "comp271-data-structures-fall2025.pdf" }
  note: ""

- code: "COMP 388/488"
  title: "Topics in CS: Network Science"
  level: "Undergraduate + Graduate"
  summary: >
    Network science as a unifying framework for interconnected systems: network
    types and representations, centralities, community detection, generative
    models, and epidemic spread (488 only). Includes a semester-long project.
  topics: [representations, centrality, community detection, generative models, epidemics]
  offerings:
    - { term: "Spring 2025", modality: "Online", syllabus: "comp388-488-network-science-spring2025.pdf" }
  note: ""

- code: "COMP 141"
  title: "Introduction to Computing Tools & Techniques"
  level: "Undergraduate"
  summary: >
    The Unix shell environment, the command line, and shell scripting for
    computer-aided problem solving. Fully online; no prerequisites.
  topics: [Unix shell, command line, shell scripting]
  offerings:
    - { term: "Fall 2024", modality: "Online", syllabus: "comp141-intro-computing-tools-fall2024.pdf" }
  note: ""
```

- [ ] **Step 4: Create `_includes/courses.html`**

```liquid
<div class="course-list">
{%- for c in site.data.courses -%}
  <div class="course-card">
    <div class="course-head">
      <span class="course-code">{{ c.code }}</span>
      {%- if c.level and c.level != "" -%}<span class="course-level">{{ c.level }}</span>{%- endif -%}
    </div>
    <h3 class="course-title">{{ c.title }}</h3>
    {%- if c.summary and c.summary != "" -%}<p class="course-summary">{{ c.summary }}</p>{%- endif -%}
    {%- if c.topics and c.topics.size > 0 -%}
    <div class="course-topics">
      {%- for t in c.topics -%}<span class="topic-chip">{{ t }}</span>{%- endfor -%}
    </div>
    {%- endif -%}
    {%- if c.offerings and c.offerings.size > 0 -%}
    <div class="course-terms">
      {%- for o in c.offerings -%}
        {%- if o.syllabus and o.syllabus != "" -%}
        <a class="term-btn" href="{{ o.syllabus | prepend: '/assets/pdf/syllabi/' | relative_url }}" target="_blank" rel="noopener">{{ o.term }}{% if o.modality and o.modality != "" %} · {{ o.modality }}{% endif %} <i class="fas fa-file-pdf"></i></a>
        {%- else -%}
        <span class="term-btn term-btn--nolink">{{ o.term }}</span>
        {%- endif -%}
      {%- endfor -%}
    </div>
    {%- endif -%}
    {%- if c.note and c.note != "" -%}<p class="course-note">{{ c.note }}</p>{%- endif -%}
  </div>
{%- endfor -%}
</div>
```

- [ ] **Step 5: Replace `_pages/teaching.md`**

```markdown
---
layout: page
title: teaching
permalink: /teaching/
nav: true
nav_order: 5
description: Courses I teach at Loyola University Chicago.
---

A selection of courses I have taught at Loyola University Chicago. Each card
links to the syllabus for every term the course was offered.

{% include courses.html %}
```

- [ ] **Step 6: Build**

Run: `bundle exec jekyll build`
Expected: exit 0.

- [ ] **Step 7: Assert teaching renders + a syllabus link resolves**

Run: `grep -c "course-card" _site/teaching/index.html`
Expected: `5`.

Run: `grep -o "comp306-406-data-mining-fall2025.pdf" _site/teaching/index.html | head -1`
Expected: a match.

Run: `test -e "_site/assets/pdf/syllabi/comp306-406-data-mining-fall2025.pdf" && echo OK`
Expected: `OK` (the renamed PDF was copied into `_site`, so the link is not a 404).

Run: `grep -o 'href="/teaching/"' _site/index.html | head -1`
Expected: a match (Teaching in nav).

- [ ] **Step 8: Commit**

```bash
git add assets/pdf/syllabi _data/courses.yml _includes/courses.html _pages/teaching.md
git commit -m "feat: add data-driven Teaching page with syllabi (web-safe slugs)"
```

---

## Task 8: Favicon from the lab logo

**Files:**
- Create: `assets/img/favicon.svg`
- Modify: `_config.yml:17`

- [ ] **Step 1: Create `assets/img/favicon.svg`** (square network-style "S", logo palette)

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <rect width="100" height="100" rx="22" fill="#0d3b40"/>
  <g stroke="#efe9d8" stroke-width="4" stroke-linecap="round" fill="none">
    <path d="M70 30 L40 30 L40 50 L60 50 L60 70 L30 70"/>
  </g>
  <g>
    <circle cx="70" cy="30" r="7" fill="#23a39a"/>
    <circle cx="40" cy="30" r="7" fill="#7378cd"/>
    <circle cx="40" cy="50" r="7" fill="#7378cd"/>
    <circle cx="60" cy="50" r="7" fill="#23a39a"/>
    <circle cx="60" cy="70" r="7" fill="#7378cd"/>
    <circle cx="30" cy="70" r="7" fill="#23a39a"/>
  </g>
</svg>
```

- [ ] **Step 2: Point `_config.yml` at it**

Find line 17:
```yaml
icon: new-logo.png # graph-icon.png  # the emoji used as the favicon (alternatively, provide image name in /assets/img/)
```
Replace with:
```yaml
icon: favicon.svg # square network-style favicon derived from the Sikdar Lab logo
```

- [ ] **Step 3: Build**

Run: `bundle exec jekyll build`
Expected: exit 0. (`_config.yml` changes require a fresh build — already doing one.)

- [ ] **Step 4: Assert the favicon link is wired**

Run: `grep -o "favicon.svg" _site/index.html | head -1`
Expected: a match.

Run: `test -e _site/assets/img/favicon.svg && echo OK`
Expected: `OK`.

- [ ] **Step 5: Commit**

```bash
git add assets/img/favicon.svg _config.yml
git commit -m "feat: add lab-logo-derived favicon"
```

---

## Task 9: About CTA + final verification

**Files:**
- Modify: `_pages/about.md`

- [ ] **Step 1: Add a "Join the lab" CTA to `_pages/about.md`**

After the final paragraph (the research-interest sentence ending in `*computational social science*.`), add a blank line and:
```markdown

<a class="btn-accent" href="{{ '/lab/' | relative_url }}">Join the Sikdar Lab →</a>
```

- [ ] **Step 2: Build**

Run: `bundle exec jekyll build`
Expected: exit 0.

- [ ] **Step 3: Assert the CTA rendered**

Run: `grep -o "btn-accent" _site/index.html | head -1`
Expected: a match.

- [ ] **Step 4: Commit**

```bash
git add _pages/about.md
git commit -m "feat: add Join the lab CTA on the about page"
```

- [ ] **Step 5: Full visual verification pass**

Run: `bundle exec jekyll serve` and open `http://localhost:4000`. Confirm:
- Loads in **dark mode on first visit**; the sun/moon toggle switches to light and the choice persists on reload.
- Nav shows exactly: **about · publications · lab · cv · teaching**; the active item has a teal underline.
- Headings/name render in **Newsreader** (serif), body in **Inter**; teal links/buttons; prose lines are not full-width.
- **Selected papers** + **news** on the homepage render correctly; a publication entry on `/publications/` (jekyll-scholar card) looks right; **MathJax** still renders math if present.
- `/lab/` shows the lab logo (correct variant per theme), four panels (PI / Collaborators / Current / Alumni), placeholder cards with **initials avatars**, badges, and link icons.
- `/teaching/` shows 5 course cards; clicking a term button opens the correct syllabus PDF (no 404).
- Favicon appears in the browser tab.
- `/projects/`, `/repositories/`, `/collaborators/` return 404.
- Spot-check contrast in **both** themes (links, badges, accent on background).

- [ ] **Step 6 (optional): open a PR**

```bash
git push -u origin website-refresh
gh pr create --title "Website refresh" --body "Editorial restyle (dark default, teal accent from lab logo), new data-driven Lab and Teaching pages, removed stale template pages. See docs/superpowers/specs/2026-06-04-website-refresh-design.md."
```

---

## Self-review notes
- **Spec coverage:** §1 typography → Tasks 1,4; §1 palette → Task 2; §2 dark default + reading width → Tasks 3,4; §3 Lab → Task 6; §4 nav + deletions + about → Tasks 5,9; §5 Teaching + syllabi rename → Task 7; §6 favicon + lab logos → Tasks 6 (logos),8 (favicon). All covered.
- **No unit-test harness** is intentional (static site); verification is build + `_site` assertions + visual pass.
- **Liquid badge indexing** by an outer-loop variable is awkward; Task 6 uses an explicit `{%- case group -%}` block to set the per-group title/badge instead.
