# ramihamada.com

Personal portfolio. Plain Jekyll, no external theme, builds on GitHub Pages.

## Structure

```
_config.yml              site settings
index.md                 front page (front matter only, no body copy)
_projects/*.md           one file per project, becomes a page
_data/experience.yml     the Experience list
_layouts/                default, home, project
_includes/head.html      meta, fonts, stylesheet
assets/css/main.scss     all styling, single file
assets/images/           project images
CNAME                    custom domain
```

## Adding a project

Drop a new file in `_projects/`. Front matter drives the layout:

```yaml
---
title: Project Name
permalink: /projects/slug/
order: 4                 # position on the front page, low numbers first
summary: One sentence. Shows on the front page and under the title.
stack: Comma, separated, tools
role: What you actually did
status: In progress
specs:                   # optional, shows as the big numbers on the project page
  - label: Sample rate
    value: 500 Hz
---

Body in Markdown.
```

Images go in `assets/images/` and are referenced like this:

```html
<figure>
  <img src="/assets/images/name.png" alt="Describe it." loading="lazy"
       width="1200" height="800">
  <figcaption>Optional caption.</figcaption>
</figure>
```

Set `width` and `height` to the file's real pixel dimensions. It stops the page
from jumping while images load. Two images side by side: wrap the figures in
`<div class="figure-pair">`.

Video embed: wrap the iframe in `<div class="video">`.

## Running it locally

```bash
bundle install
bundle exec jekyll serve --livereload
```

Then open http://localhost:4000.

## Deploying

Push to `master`. GitHub Pages builds it. Check
Settings > Pages to confirm the source branch.
