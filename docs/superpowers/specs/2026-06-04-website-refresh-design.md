# Website Refresh — Design Spec

**Date:** 2026-06-04
**Site:** satyakisikdar.github.io (satyaki.net) — al-folio Jekyll theme on GitHub Pages
**Scope:** Visual restyle + content cleanup + new Lab page + data-driven Teaching page + identity tweaks. Restyle *within* al-folio (retarget SCSS variables, no engine replacement).

---

## 0. Goals & non-goals

**Goals**
1. Distinctive editorial look (serif headings + sans body), **dark mode as default**, soft/neutral palette on a cool charcoal base with a single **teal** accent drawn from the SIKDAR LAB logo.
2. Remove stale al-folio template pages.
3. New **Sikdar Lab** page driven by `_data/lab.yml` (PI / Collaborators / Current / Past).
4. New **Teaching** page driven by `_data/courses.yml`, with course summaries + linked syllabus PDFs.
5. Identity: theme-matched favicon.

**Non-goals (do not touch):** blog/distill system, plugins, analytics config, publications data (`papers.bib`), deploy pipeline, `_data/coauthors.yml` (separate system — drives bibliography author hyperlinks), `_data/venues.yml`, news collection.

---

## 1. Palette & typography

### Color tokens

Accent is **teal**, derived from the SIKDAR LAB logo (the "LAB" nodes). Single-accent / restrained ("Option 3"): teal only in the UI; the logo itself supplies the violet, so **violet is reserved for the logo and not used as a UI color.** Base is a slightly cool charcoal to sit under the logo.

| token | dark (default) | light (toggle) | use |
|---|---|---|---|
| bg | `#1a201f` | `#f6f7f5` | page background |
| surface / card | `#222a28` | `#ffffff` | cards, panels |
| rule | `#33403d` | `#e2e5e1` | borders, dividers |
| ink (text) | `#e7e6e1` | `#21302d` | body text |
| muted | `#92978f` | `#5f6b66` | secondary text |
| **accent (teal)** | `#27a89e` | `#1d7a72` (darkened for contrast on paper) | links, heading highlight, buttons, active nav |
| accent-soft | teal @ ~16% alpha | teal @ ~14% alpha | badge backgrounds, hovers |
| avatar/neutral | slate `#46514f` | slate `#cfd6d2` | initials-avatar background |
| logo violet | `#7378cd` | `#7378cd` | **logo only** — reserved, not a UI accent |

Logo colors for reference (do not invent): SIKDAR nodes ≈ violet `#7378cd`, LAB nodes ≈ teal `#23a39a`, edges ≈ cream `#efe9d8`, logo plate ≈ dark teal `#0d3b40`.

Implementation: map these onto al-folio's existing CSS custom properties in `_sass/_themes.scss` (`--global-bg-color`, `--global-text-color`, `--global-theme-color`, `--global-hover-color`, `--global-card-bg-color`, `--global-divider-color`, etc.) for both `:root` (light) and `html[data-theme='dark']`. Update raw SCSS color vars in `_sass/_variables.scss` as needed (the file currently holds a half-applied solarized palette — replace the active `$*-color` assignments that feed the theme).

### Typography
- **Headings + site name:** `Newsreader` (serif).
- **Body / UI:** `Inter`.
- In the Google Fonts `<link>` in `_includes/head.html` (currently `Noto+Sans:...|Material+Icons`), swap `Noto+Sans` → `Newsreader` + `Inter`. **Keep `|Material+Icons` — al-folio uses it for UI glyphs (theme toggle, etc.); removing it breaks icons site-wide.**
- In `_sass/_base.scss`: set body `font-family` to Inter, headings/name to Newsreader; **remove the `font-weight: 500 !important` override** (line ~8/24) so weights are controllable.
- Constrain prose line length: cap main text column at ~`760px` (e.g. on `.post`, page content containers) while leaving the `max_width: 1100px` page container (in `_config.yml`) intact for nav/cards/grids.

**Files:** `_sass/_variables.scss`, `_sass/_themes.scss`, `_sass/_base.scss`, `_sass/_layout.scss`, `_includes/head.html`.

---

## 2. Dark mode as default

`assets/js/theme.js` → `initTheme()` (runs in `<head>`, lines ~90–101). Currently: no stored choice → fall back to `prefers-color-scheme`, which yields light unless the OS is dark.

Change: when there is **no stored choice** (`theme == null || "null"`), default to `"dark"`. Still:
- honor an explicit prior user toggle saved in `localStorage`,
- remain fully toggle-able via the existing sun/moon control.

Keep it in `<head>` (no flash). Verify the highlight-theme swap (`setHighlight`) and table-dark classes still behave with dark as the initial state.

---

## 3. Lab page (primary feature)

### Data: `_data/lab.yml`
Grouped by role; every field optional except `name`. Placeholders only — no invented real people.

```yaml
pi:
  - name: Satyaki Sikdar
    role: Principal Investigator
    image: satyaki-2024-bw.png        # in assets/img/; omit → initials avatar
    bio: >
      One-paragraph bio (reuse about-page voice).
    links:
      website: https://satyaki.net
      scholar: https://scholar.google.com/citations?user=2f6t6hYAAAAJ
      github:  https://github.com/satyakisikdar

collaborators:                          # other faculty — PLACEHOLDERS
  - name: Collaborator Name
    role: "Professor, Dept · University"
    image:
    bio: >
      One line. Placeholder — fill in.
    links: { website: "" }

current:                                # students — PLACEHOLDERS
  - name: Student Name
    role: PhD Student
    since: 2025
    image:
    bio: >
      Research interests. Placeholder.
    links: { github: "" }

past:                                   # alumni — PLACEHOLDERS
  - name: Alum Name
    role: "MS '25"
    now: "Now at Company / School"
    image:
    links: { website: "" }
```

### Page: `_pages/lab.md`
- `layout: page`, `permalink: /lab/`, `title: lab`, `nav: true`, `nav_order` placing it after publications.
- **Header shows the SIKDAR LAB logo** — `assets/img/lab-logo-light.png` and `lab-logo-dark.png` (both already in repo), swapped per theme using al-folio's existing `repo-img-light` / `repo-img-dark` class pattern (see `_sass/_themes.scss`). Center it above the intro.
- Intro blurb (lab name + research focus) + a short "Join the lab" note for prospective students (with contact/CTA).
- Body renders the include below.

### Include: `_includes/lab_members.html`
- Iterate the four groups in order: **PI → Collaborators → Current Students → Alumni**, each as a titled panel.
- Each member = a card: photo (or **initials avatar** on slate bg when `image` empty), name, role, optional `since`/`now`, bio, and link icons (website/scholar/github → reuse al-folio's social icon set / academicons).
- Graceful degradation: missing fields render nothing (no broken markup); a half-filled `lab.yml` still looks complete.
- Badges: PI = teal solid · Current = teal-soft · Alum = neutral grey.
- Responsive grid (cards wrap; reuse al-folio/Bootstrap grid utilities already loaded).

Self-contained: no dependency on `coauthors.yml`.

---

## 4. Content & structure

### Navigation (final)
`about · publications · lab · cv · teaching`

### Delete
- `_pages/projects.md` (template placeholder)
- `_pages/repositories.md` (template placeholder)
- `_pages/collaborators.md` (personal TODO scratchpad)
- `_data/repositories.yml` (only consumed by the deleted repositories page — confirm no other reference before removing)

Verify nothing else `include`s the deleted pages' partials; if `_includes/projects.html` / `repositories.*` become orphaned, leave them (harmless) or remove if clearly unused.

### About (`_pages/about.md`)
Keep the real bio verbatim. Light touch only: ensure it renders well under the new style; add a "Join the lab" CTA line linking to `/lab/`. No invented facts. `selected_papers` + `news` keep working.

---

## 5. Teaching page (data-driven + syllabi)

### Web-safe syllabus rename
Rename the originals in `assets/pdf/syllabi/` (spaces + a colon break URLs). Mapping:

| original | web-safe slug |
|---|---|
| `Fall 2024 - Comp 141 Combined.pdf` | `comp141-intro-computing-tools-fall2024.pdf` |
| `Fall 2024 - COMP 406.pdf` | `comp406-data-mining-fall2024.pdf` |
| `Spring 2025 - COMP 306-406.pdf` | `comp306-406-data-mining-spring2025.pdf` |
| `Fall 2025 - COMP 306-406.pdf` | `comp306-406-data-mining-fall2025.pdf` |
| `Spring 2026 - COMP 306-406.pdf` | `comp306-406-data-mining-spring2026.pdf` |
| `Fall 2025 - COMP271 SyllabusV6.pdf` | `comp271-data-structures-fall2025.pdf` |
| `Spring 2025 - COMP 388:488 Net Sci.pdf` | `comp388-488-network-science-spring2025.pdf` |
| `Spring 2026 - HONR 204-03H.pdf` | `honr204-science-society-network-science-spring2026.pdf` |

Also remove `assets/pdf/syllabi/.DS_Store`.

### Data: `_data/courses.yml`
One entry per course; multiple `offerings` (term + modality + syllabus slug). `summary`/`topics` pre-filled from syllabi as **editable starting content** (final wording TBD by author).

```yaml
- code: "COMP 306/406"
  title: "Data Mining"
  level: "Undergraduate + Graduate"
  summary: >
    Theory and practice of analyzing very large datasets: structured and
    unstructured data, ETL with Pandas, cleaning, visualization, frequent-
    pattern mining, clustering, and supervised learning. The 406 (graduate)
    section adds research components and formal writing.
  topics: [Pandas ETL, data cleaning, visualization, frequent-pattern mining,
           clustering, supervised learning]
  offerings:
    - { term: "Fall 2025",   modality: "In person", syllabus: "comp306-406-data-mining-fall2025.pdf" }
    - { term: "Spring 2025", modality: "In person", syllabus: "comp306-406-data-mining-spring2025.pdf" }
    - { term: "Fall 2024",   modality: "Online",    syllabus: "comp406-data-mining-fall2024.pdf" }
    - { term: "Spring 2026", modality: "In person", syllabus: "comp306-406-data-mining-spring2026.pdf" }
  note: ""
```

Full course set to seed (summaries are starting drafts, author to finalize):

| code | title | level | terms |
|---|---|---|---|
| COMP 141 | Introduction to Computing Tools & Techniques | Undergraduate | Fall 2024 (online) |
| COMP 271 (/400B) | Data Structures I | Undergraduate (+ grad 400B) | Fall 2025 |
| COMP 306/406 | Data Mining | Undergrad + Graduate | F24 (as 406, online), S25, F25, S26 |
| COMP 388/488 | Topics in CS: Network Science | Undergrad + Graduate | Spring 2025 |
| HONR 204 | Science & Society: Network Science | Honors | Spring 2026 |

Seed summaries (draft):
- **COMP 141** — Unix shell environment, command line, and shell scripting for computer-aided problem solving; fully online. No prerequisites.
- **COMP 271** — Data abstraction and structures (stacks, queues, lists, sets, trees), ADTs, recursion, sorting/searching, correctness & efficiency. Programming-intensive (Python). Prereqs: COMP 170, COMP 163.
- **COMP 306/406** — (see YAML above).
- **COMP 388/488** — Network science as a unifying framework: network types & representations, centralities, community detection, generative models, and epidemic spread (488 only). Includes a semester-long project.
- **HONR 204** — Honors seminar: measuring network structure, identifying key nodes/communities, modeling contagion; theory plus real-world examples across CS, sociology, political science, and biology.

### Page + include
- `_pages/teaching.md`: `permalink: /teaching/`, `nav: true`, replaces current template text with a short intro + the include.
- `_includes/courses.html`: each course = a card (code · title · level badge, summary, topic chips, and a row of term buttons each linking to its syllabus PDF via `assets/pdf/syllabi/<slug>`). Sort courses by most recent offering.

---

## 6. Identity
- **Favicon:** derive from the SIKDAR LAB logo, not an invented monogram. Produce a **square** favicon echoing the logo's network style — a small teal+violet node cluster (or the network-style "S" glyph) on the dark-teal plate `#0d3b40`. Save to `assets/img/` and point `icon:` in `_config.yml` at it, replacing `new-logo.png`. (The existing wide wordmark PNGs are not square, so a purpose-made square crop/derivative is needed.)
- **Lab logos:** `assets/img/lab-logo-light.png` + `lab-logo-dark.png` (supplied) are used on the Lab page header (see §3), theme-swapped.
- **Profile photo:** keep `satyaki-2024-bw.png` (suits the palette). No change.
- **Footer:** unchanged (al-folio attribution stays per theme license).

---

## 7. Verification
Mockups (visual companion) only approximate the look. The real check is a local render:
1. `bundle exec jekyll serve` (or `docker compose up` if native deps fail).
2. Confirm, in browser: dark default on first load; toggle to light works and persists; nav shows the 5 pages; serif headings + Inter body; teal accent; reading column width; selected-papers + news + jekyll-scholar publication cards styled correctly; MathJax still renders; Lab page panels render with placeholders + initials avatars; Teaching cards link to the renamed syllabus PDFs (no 404s); favicon shows; deleted pages 404.
3. Spot-check both themes for contrast (accent on bg, links, badges).

## 8. Workstream order (for the plan)
1. Palette + type tokens (SCSS + head.html) — foundation.
2. Dark default (theme.js).
3. Nav + deletions (low risk).
4. Lab page (data + page + include).
5. Teaching (rename syllabi + courses.yml + page + include).
6. Favicon.
7. About CTA + reading-width polish.
8. Full `jekyll serve` verification pass.
