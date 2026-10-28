# Deniz – 2nd Grade Reading & Writing Training

## Purpose
Resources to assess and improve Deniz's reading and writing. Deniz is in Year 2 at a
public primary school in Sydney (NSW curriculum). English is her second language. She
speaks and listens well in English, but her teacher has reported that she is well behind
in reading and writing. The goal is to bring her reading and writing up to the expected
Year 2 level.

## Workflow (one loop per session)
1. Plan the next session in chat (assessment or training).
2. Build `docs/sessions/session-NN.html`: one self-contained page, about 30 minutes of work.
3. Deniz works through it on the parent's phone, with the parent next to her.
4. The last screen exports `session-NN-result.json` (Web Share API, with a download fallback).
5. The parent uploads the JSON in chat. Save it to `data/results/session-NN-result.json`,
   update `docs/progress/progress.json` (skill statuses and evidence), and plan the next session
   around the gaps.

Schedule: daily sessions.
- Sessions 1–3 are the baseline assessment:
  - S1: letter sounds, phonics, high-frequency words
  - S2: reading aloud and comprehension
  - S3: spelling and writing
- After the baseline, every 5th session is a short re-check. The rest are training sessions.

## Target device and design rules
- **Samsung Galaxy A52s 5G**, Chrome on Android, portrait. CSS viewport is about 412 × 915.
  Design for widths 360–430 px. No desktop or print layouts and no printable material.
- Mobile first: `<meta name="viewport" content="width=device-width, initial-scale=1">`,
  layouts in a single column, nothing that scrolls sideways.
- Tap targets at least 48 px. Body text at least 18 px; words being read are 28–40 px.
- One task per screen. Show a step progress indicator. Fixed bottom action bar for Next/Back.
- Child-friendly: lots of white space, very short instructions, friendly encouragement
  (no red crosses or "wrong" shown to the child). English only (no first-language support).
- 🔊 button on instructions and on dictated words, using `speechSynthesis` (prefer an
  `en-AU` voice, fall back to any `en-*` voice).
- **Typing** is the writing mode. Use inputs with `autocomplete="off" autocorrect="off"
  autocapitalize="off" spellcheck="false"`. The parent setup screen reminds the parent to
  turn off Samsung Keyboard predictive text and auto-correct.
- **Parent-scored items** (reading aloud, fluency) use a clearly marked parent panel with
  large ✓ / ✗ / skip buttons.
- Keep the screen awake (`navigator.wakeLock`, wrapped in try/catch).
- Autosave progress to `localStorage` (wrapped in try/catch), so an accidental reload resumes
  where she was.
- Self-contained: inline CSS and JS, no frameworks. Google Fonts are optional, with system
  font fallbacks.
- **Every time a new session page is built**, update BOTH `next_session` in `docs/progress/progress.json` and the
  fallback `href` and "Session N" label in `docs/index.html`, so the child's Start button always opens the latest session.

## Session page structure
1. Parent setup screen: keyboard tips, volume on, "hand the phone to Deniz".
2. Warm-up (~3 min): easy items for a confidence win.
3. 2–3 main blocks (6–8 min each), with a short break screen between them.
4. Finish: Deniz rates how it felt (emoji), plus an optional parent note, then Export.

## Result JSON (schema `deniz-session/1`)
```json
{
  "schema": "deniz-session/1",
  "session": 1,
  "kind": "assessment | training | recheck",
  "title": "Baseline 1 – sounds and words",
  "started_at": "ISO-8601",
  "finished_at": "ISO-8601",
  "duration_sec": 0,
  "device": "navigator.userAgent",
  "steps": [
    {
      "id": "s1-digraphs",
      "skills": ["ph2"],
      "mode": "auto | parent",
      "started_at": "ISO-8601",
      "duration_sec": 0,
      "items": [
        { "prompt": "ship", "expected": "ship", "response": "sip",
          "correct": false, "attempts": 1, "ms": 4200, "skipped": false }
      ],
      "score": { "correct": 0, "total": 0 },
      "notes": ""
    }
  ],
  "child_feeling": "great | ok | hard",
  "parent_note": ""
}
```
- Optional extra fields (added in Session 1, additive, so the schema version stays `deniz-session/1`):
  `parent_context` (home reading, first-language reading, vision/hearing, school info),
  `observations` (parent-tapped behaviour chips), `summary` (letters, per-group and per-tier
  scores, indicative level, `flags`, `suggested_progress`, `evidence`, `not_reached`).
  Items may carry `quality` (`quick | slow | wrong` for sight words), `score` (0, 0.5 or 1 for
  meaning items), `skills` (item-level skill IDs) and `notShown: true` (skipped by an
  early-stop rule, so exclude from totals). A step may carry `stopped_early`.
- Early-stop rules in Session 1: a phonics group stops after 4 shown words with 1 or fewer
  correct; phonics halts after 3 stopped groups in a row; made-up words are skipped if the
  first group stopped; a sight-word tier stops after 10 words with 2 or fewer correct, and
  later tiers are then skipped.
- Every step lists the skill IDs it gives evidence for. Skill IDs come from
  `docs/curriculum/year2-reading-writing.html` (ph1–ph12, fl1–fl7, co1–co13, wr1–wr22,
  sp1–sp15, hw1–hw6, vo1–vo7, li1–li5, ol1–ol4).
- Keep timing data: it gives evidence for automaticity.

## Progress (`docs/progress/progress.json`, schema `deniz-progress/1`)
- One entry per skill: `status` (`not_checked` | `not_yet` | `learning` | `secure`) and
  an `evidence` list of `{ "session": N, "correct": x, "total": y, "note": "" }`.
- Rough guide: below 50% correct = `not_yet`; 50–89% = `learning`; 90% or more,
  reasonably quick, and seen in 2 sessions = `secure`.
  Adjust using the parent's notes.
- Every time results are processed, append the session number to `sessions_completed`.

## Repository rules
- This repo is the single home for everything in this project.
- Deliverables are static HTML in `docs/` (published with GitHub Pages: branch `main`,
  folder `/docs`). Data lives in `data/`. The repo is public, so use only Deniz's first
  name: no school name, photos or other identifying details.
- Australian English spelling (colour, favourite, Mum).
- `docs/` layout: `curriculum/` (reference), `sessions/` (one page per session).

## Git
- Default branch: `main`. Remote: GitHub (`janfeshanm/deniz_2nd_Training`).

## Scoring rules (from Session 2)
- A skipped or "couldn't read" item counts as **wrong** and stays in the totals. Only items never shown
  because of an early-stop rule (`notShown: true`) are excluded. (Session 1 results excluded skips, so its
  sight-word, trigraph and less-common-spelling totals are slightly generous.)
- Parent ✓ / ✗ buttons are disabled for about 1.8 s after each word appears, so taps reflect real reading.
- Session 2 adds step-level fields on passage steps: `level`, `words`, `errors`, `accuracy`, `seconds`, `wcpm`,
  `self_corrections`, `behaviours`. Comprehension items carry `kind` and `modality` (`reading` or `listening`);
  listening evidence is kept but not used for a skill's status.
- `docs/progress/progress.json` also holds `school_context` (EAL/D phase and levels only: no names).
- **Voice recordings** (Session 2 on): the page can record Deniz reading each story and exports the audio files
  next to the result JSON (`session-NN-<step>.webm`). Audio is never committed: the repo is public. Keep it in
  `data/audio/` (git-ignored). Do not put the EAL/D report or other school documents in the repo; only extracted
  levels go in `progress.json` (`school_context`).
