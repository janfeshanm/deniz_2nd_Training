# Deniz – 2nd Grade Reading & Writing Training

## Purpose
Resources to assess and improve Deniz's reading and writing. Deniz is in Year 2 at a
public primary school in Sydney (NSW curriculum). English is her second language. She
arrived in Australia about 16 months ago (mid-2025). Her speaking and listening are good,
but her teacher has reported that she is well behind in reading and writing. The goal is
to bring her reading and writing up to the expected Year 2 level.

## Repository rules
- This repo is the single home for everything in this project: assessments, lesson
  plans, worksheets, word lists, progress records and notes.
- **Every deliverable is a static HTML page in `docs/`.** No build step, no frameworks
  or server code. Each page is a self-contained `.html` file with inline CSS and JS.
  Only reference other local files with relative paths.
- `docs/index.html` is the home page. Link every new deliverable from it.
- Pages should print cleanly on A4 (`@media print`), because worksheets get printed.
- Use child-friendly design: large, clear fonts (e.g. Andika, Lexend or a system sans
  font), plenty of white space, and simple instructions written for a 7–8-year-old.
- Australian English spelling (colour, favourite, Mum).
- Suggested layout inside `docs/`:
  - `assessment/` – baseline and periodic checks (phonics, sight words, fluency, writing samples)
  - `lessons/` – teaching plans and activities
  - `worksheets/` – printable practice sheets
  - `progress/` – tracking pages
  - `assets/` – shared images or audio, if needed

## Git
- Default branch: `main`. Remote: GitHub (`janfeshanm/deniz_2nd_Training`).
- `docs/` can be published with GitHub Pages (Settings → Pages → branch `main`, folder `/docs`).
