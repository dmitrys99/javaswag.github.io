# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Javaswag is a Russian-language podcast website about Java development, hosted at https://javaswag.github.io. It's built with Hugo (static site generator) and deployed to GitHub Pages. The site contains 100+ episodes with audio hosted on Yandex Cloud S3.

## Build & Preview Commands

```bash
# Install Hugo (macOS)
brew install hugo

# Build site locally (includes drafts, builds to docs/, runs PostCSS)
make build

# Preview built site at http://localhost:1313/
make preview

# Build and preview in one step
make run

# Install npm dependencies (PostCSS pipeline)
npm install

# CSS only: build with PostCSS (import, autoprefixer, cssnano)
npm run build

# Lint CSS
npm run lint
```

Hugo version used in CI: **0.145.0** (extended).

## Architecture

### Hugo Site Structure

- **config.yml** — Hugo configuration. Audio files served from `params.yandexS3` (Yandex Cloud S3 bucket). Site language is Russian (`ru`).
- **layout/** — Hugo templates (note: not the default `layouts/` directory, configured via `layoutdir` in config.yml)
  - `index.html` — Homepage listing episodes with play buttons
  - `_default/episode.html` — Individual episode page with waveform audio player
  - `_default/draft.html` — Draft episode layout
  - `rss.xml` and `draft/rss.xml` — Podcast RSS feeds (iTunes-compatible)
  - `partials/` — Reusable template fragments (header, footer, meta, script)
- **content/episode/{N}.md** — Episode markdown files (numbered 0-100+). Front matter includes: layout, number, title, date, people, audio filename, waveform, size, guid, duration, draft status.
- **content/draft/** — Draft episode content
- **content/about/**, **content/guests/**, **content/notes/**, **content/captions/** — Supporting content sections
- **data/people.yml** — Guest/host registry mapping person slugs to URLs and avatar images. Referenced by episode front matter `people` field.
- **static/** — Static assets (images, waveform binary files, Waveform JS player)
- **assets/css/** — Source CSS processed by PostCSS pipeline
- **docs/** — Build output directory (gitignored, served by GitHub Pages)

### Go Utilities

- **exporter/** — Go tool that generates episode markdown files by fetching the SoundCloud RSS feed and Yandex S3 audio listing, then merging them into Hugo content files.
- **diff/** — Go tool that compares the SoundCloud RSS feed against the built `docs/index.xml` to detect differences. Runs in CI after Hugo build.
- **audio/** — Separate Go application ("waveform") for MP3 upload and waveform binary generation. Deployed independently to Fly.io. Has its own [CLAUDE.md](audio/CLAUDE.md).

### CI/CD

GitHub Actions workflow (`.github/workflows/gh-pages.yml`): builds with Hugo + npm, runs the diff tool, and deploys `docs/` to GitHub Pages via `peaceiris/actions-gh-pages`.

### Episode Content Format

Episodes follow this front matter pattern:
```yaml
layout: episode
number: 1
title: "#1 - Guest Name - Topic"
date: 2019-07-25 13:43:15
people:
  - volyx
  - guest-slug
audio: 1-javaswag-guest-name.mp3
waveform: standard.avg220.bin
size: 201693492
guid: tag:soundcloud,2010:tracks/655299722
image: images/logo.png
description: Short description
duration: 01:24:07
draft: false
```

Person slugs in `people` must match keys in `data/people.yml`. Audio files are not stored in the repo — they're served from `https://storage.yandexcloud.net/javaswag/`.
