# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal academic website for Satyaki Sikdar (https://satyaki.net), built on the
[al-folio](https://github.com/alshedivat/al-folio) Jekyll theme and hosted on GitHub Pages.
Content is mostly Markdown/YAML/BibTeX; layout is Liquid + SCSS. There is no application code to
test — "development" means editing content and templates, then previewing the static build.

## Commands

```bash
bundle install                       # install Ruby gems (first time / after Gemfile change)
bundle exec jekyll serve             # local dev server at http://localhost:4000, live-reload
bundle exec jekyll serve --lsi       # same, with related-posts (LSI) generation — slower
bin/cibuild                          # production build: `bundle exec jekyll build --lsi`
bin/deploy                           # build + purgecss, push to gh-pages branch (prompts first)
docker compose up                    # alternative: run the site in a container (port 8080)
```

There is no test suite or linter beyond pre-commit hooks (`.pre-commit-config.yaml`:
trailing-whitespace, end-of-file-fixer, check-yaml, check-added-large-files). Run `pre-commit run
--all-files` if pre-commit is installed.

Deployment is automatic: GitHub Actions builds `master` and publishes. `bin/deploy` is the manual
fallback. Do not hand-edit the `gh-pages` branch — it is generated.

## Where content lives (edit these, not the HTML output)

- `_config.yml` — single source of truth for site config: identity, social handles, enabled
  features (dark mode, math, analytics), `max_width`, jekyll-scholar settings, plugin list.
  Changing `_config.yml` requires restarting `jekyll serve` to take effect.
- `_pages/about.md` — the homepage (`permalink: /`). Front matter toggles `news`,
  `selected_papers`, profile image/address.
- `_pages/*.md` — top-level nav pages: `cv`, `publications`, `projects`, `teaching`,
  `repositories`, `collaborators`, `dropdown`. Nav order/visibility is set per-page in front
  matter (`nav`, `nav_order`).
- `_bibliography/papers.bib` — all publications. jekyll-scholar renders these. Per-entry custom
  BibTeX fields drive the UI: `selected={true}` surfaces a paper on the homepage; `abbr`, `pdf`,
  `arxiv`, `code`, `slides`, `html`, `preview` (thumbnail), `bibtex_show`, `abstract`. The list in
  `filtered_bibtex_keywords` (in `_config.yml`) hides these custom fields from the shown BibTeX.
- `_news/*.md` — news/announcement items (a Jekyll collection). The homepage shows the most recent
  `announcements.limit` (currently 7).
- `_data/*.yml` — structured data: `venues.yml`, `repositories.yml` (GitHub repo cards),
  `coauthors.yml`.
- `assets/` — `img/`, `pdf/` (CV lives here), `css/`, `js/`, `json/`, etc. Profile image and
  favicon (`icon:` in `_config.yml`) are under `assets/img/`.

## Layout & styling architecture

- `_layouts/` — page templates. `default.html` is the shell (head/nav/footer); `about.html`,
  `cv.html`, `bib.html` (publication entry), `distill.html` (long-form posts), `page.html`,
  `post.html` build on it. A page's `layout:` front matter selects one.
- `_includes/` — reusable Liquid partials pulled into layouts (`header.html`, `footer.html`,
  `news.html`, `selected_papers.html`, `social.html`, `projects.html`, etc.). Edit these to change
  shared UI fragments.
- `_sass/` — styles. `_themes.scss` defines light/dark color variables; `_variables.scss`,
  `_base.scss`, `_layout.scss`, `_cv.scss`, `_distill.scss` for the rest. The compiled entry point
  is `assets/css/main.scss`. Theme colors and the dark-mode palette are changed here, not in HTML.

## Custom Jekyll plugins (`_plugins/`)

These run at build time and add Liquid tags/filters beyond stock al-folio:

- `cache-bust.rb` — appends MD5 query strings to asset URLs for cache busting.
- `details.rb` — `{% details %}…{% enddetails %}` collapsible block tag.
- `external-posts.rb` — fetches blog posts from RSS feeds listed under `external_sources`.
- `file-exists.rb` — `{% file_exists path %}` returns true/false in templates.
- `hideCustomBibtex.rb` — strips `filtered_bibtex_keywords` from displayed BibTeX.

## Gotchas

- jekyll-scholar, jekyll-imagemagick, and several plugins need native deps (ImageMagick,
  Node via mini_racer). If a local build fails on these, prefer `docker compose up`.
- `_config.yml` is excluded from live-reload; restart the server after editing it.
- Drafts and content edits hot-reload; plugin (`.rb`) and `_config.yml` changes do not.
