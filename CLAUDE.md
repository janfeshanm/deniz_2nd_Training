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

## Session 3 additions
- No early-stop cascades: a group may stop on its own (0 of first 3 correct), but it never causes later groups to be skipped.
  (Session 2 skipped every later phonics group after two stopped groups; avoid that.)
- Spelling items carry `sentence`, `group`, `first_key_ms` and `deletes` (delete key presses while typing).
- Writing steps use `mode: "writing"` and have `score: null` at step level (not auto-scored; Claude reviews the text).
  Writing items carry `kind` (`dictation | picture | recount`), `stats` (word count, capitals, full stops, lowercase "i",
  joining words; for dictation also `words_right`, `words_expected`, `misspelled`, `exact`) and, for dictation, `score`
  (fraction of words right). The recount also has `planned_aloud`, `used_starter`, `edited_after_reading`, `first_text`.
- `parent_context.predictive_off`: if not "yes", treat typed spellings with caution.
- **‹ Back button** (Session 3 on, every future page): a small button in the header undoes the last answer or goes back one
  screen (restoring typed text). The phone's back gesture does the same instead of leaving the page. "Start over" on the
  resume screen is a small link with a double confirm.

## Training sessions (Session 4 on)
- `kind: "training"`, plus `focus` (skill IDs). After a mistake the page shows the answer and she tries again.
  `correct` is always her FIRST try (used for status); retries are kept as `retry_response` / `retry_correct` (spelling)
  or `responses` / `correct_final` (tricky words). Teaching steps use `mode: "teach"` and are not scored.
- Repeated reading: one passage step with `read1` (cold, scored: items = read 1) and `read2` (after the phone reads it),
  each with errors, accuracy, seconds, wcpm, misread. Recordings `session-NN-read-<key>-1/2.webm`.
- Typing screens: the Done button sits right under the text box, and the bottom bar hides while the keyboard is open
  (it covered the box on the phone in Session 3).
- About 20–25 minutes per training session (31–33 minutes tired her).
- Session 5 adds: a 1-minute **keyboard game** (`type: "keys"`, one text box kept open; items carry `ms` to the right key and
  `attempts`; track the median over sessions for hw6), a **cold check** of the previous session's pattern before any re-teaching
  (`sN-cold-read` / `sN-cold-spell`, the retention evidence), tap-the-spelling **review** of earlier tricky words (no typing), and
  **word-tile sentence building** (`type: "build"`) to practise "and" without typing. Writing steps can prefill a sentence start.
- Session 6 adds **phrase reading** (a `read` step with `phrase: true`; ✓ = read as one smooth chunk; evidence for fl5) and a
  rule screen (ai in the middle, ay at the end). Feedback hints follow the pattern taught (magic e, ai/ay, phrase).
  Recordings live only in memory: if the page reloads they are lost, and the wrap-up and `summary.flags` now say so.
- Each training session opens with a **cold check** of the previous session's pattern and tricky words (no reminder first).
- **Curriculum page** (`docs/curriculum/year2-reading-writing.html`) reads every skill's status and evidence from
  `docs/progress/progress.json` (no manual or localStorage statuses). Keeping progress.json up to date after each session
  is enough; the skill IDs in the page and in progress.json must stay identical (91 skills).
- Session 7 adds a **long-or-short check** (`sN-check`: mixed pairs like fun/fine; `summary.check.short_wrong` = short words
  given a long sound) and a data-driven rule screen (`RULE`). Tricky words now come in families (could / would / should).

## Re-checks (Session 8 on, every 5th session)
- `kind: "recheck"`, about 20 minutes, **no teaching and no hints**: steps carry `check: true` (no feedback screens, no spelling
  retry), and the story is read once with no model reading (`single: true`, `read2: null`). Spelling items carry `before`
  (her attempt at the same word in the baseline) for a direct before/after comparison.
- **Voice picker** (Session 8 on): the setup screen lists the phone's English voices with an f/s test ("fish, sun, fan, sock,
  feet, seat"). The choice is saved in localStorage `deniz-voice` and shared by all later pages; `parent_context.voice` records it.
  Some phone voices make f sound like s (S7: feet typed as "seat"), so treat f/s spelling errors with caution.
- Keyboard-game letters are said with a word ("f, as in fish"); items carry `as`.
- Session 9 adds a **pattern sort** (`pick` step with `feedback: "pattern"` and `q`: hear a word, tap its vowel pattern; no typing)
  followed by **mixed no-cue spelling** with retry, and a tricky-word speed round (`read` with `trickyRead: true`). New pages are
  built from the latest page (S8 onward keeps the voice picker); training pages set `check` off and `single` off.
- Session 10 is a **short** session (about 20 min): no keyboard game and no phrases; writing comes early in a two-line
  `First \nThen ` frame (summary counts `lines` and `lines_with_full_stop`); mixed spelling runs with **no pattern sort first**
  to test whether choosing the pattern is becoming automatic; new pattern "all" (ball, tall, small, with bell as a contrast).
- Session 11 puts the **story first** (Deniz asked for reading first; the cold read is then fresh). The story uses no new
  pattern. Teach items may carry `html` (custom highlight) so tricky words like there/where aren't shown as magic e;
  tricky-word families are taught by adding a letter (here→there/where, hen→when, hat→what). "all" and "ar" are logged under
  ph6 (the /or/ and /ar/ sound families).
- From Session 12, pages **keep everything until the parent clears it**: answers in localStorage, recordings in IndexedDB
  (`deniz-audio`, one entry per take and read). ‹ Back, Start over and reloads never delete: replaced answers go to `S.redo`,
  every story read to `S.readLog` (take number), page openings to `S.loads`, wiped tries to `S.archive`; all exported as
  `attempt_history`, recordings named `…-take<N>-<read>.webm`. After export a "Clear this session from the phone" button
  asks for confirmation, then removes the localStorage key and that session's recordings. When evaluating, judge from
  attempt_history: a story with more than one take is not a cold read.
- From Session 13 the top bar has a **⏸ Pause** button. While paused, `Date.now()` stands still (it is wrapped), so every
  item time, the story timer and writing time leave the pause out; running recordings are paused and resumed too.
  Pauses are logged in `attempt_history.pauses` and the result has `paused_sec` / `active_sec`.
- Session 14 is a one-off **Year 1 check** (kind `y1check`), done after the Session 13 re-check: 40 words in the style of
  the NSW Year 1 Phonics Screening Check (20 real, 20 made-up "alien" words; items carry `nonsense` and `gpc`), endings and
  two-syllable words, 30 Year 1 high-frequency words, Year 1 spelling rules, one written question (wr17, scored by hand).
  The NSW check's expected level (28/40) is shown for reference only.
- Session 15 (training) is built from the Session 14 page, so training pages keep the pause button, alien-word support
  in `read` steps (items with `alien` show 👽 to her and the pronunciation to the parent) and keep-everything storage.
  Adds "igh" to the pattern highlights, `ed: true` on a read step (feedback: cover the e d), and a writing frame that fades
  (only "First " given). Short-vowel pairs (hot/hut, hill/hall) are logged under ph1, short-vowel blend words under ph3.
- `docs/vocab.html`: flip cards for the Year 2 word list. It reads `docs/voc.md` (table: Word | Meaning | Example) at load time,
  with an embedded copy as a fallback (re-embed after editing voc.md if the page is opened as a file). Ticks are kept in
  localStorage `deniz-voc-v1` (this phone only, not sent in results). Linked from the home page as "📖 Word cards".
- Session 16 (training) is built from the Session 15 page. Adds "ew" and "ue" to the pattern highlights (`TEAMS` regex,
  `teamSays`), logged under ph5 (vowel teams, as in the S14 check). The writing frame keeps "First " and adds a ➕ Then button
  (`thenBtn: true`) that starts the second sentence on a new line; the item records `used_then_btn`. The cold check mixes the
  previous session's patterns (igh, -ed) with short u / a by ear.
- Speech-to-text needs the whisper model in `data/models/` on the computer (it was missing for S15); without it, the audio
  file gets a pause analysis only (ffmpeg silencedetect).
- Session 17 (training, last before Re-check 3) is built from the Session 16 page. Adds "ou" (logged under ph7, diphthongs,
  as in the S14 check) with a rule screen (ou in the middle, ow at the end), a second writing button ➕ At the end (`endBtn`,
  `used_end_btn`) for three sentences, and the look-cover-write `tricky` step again (here, were; sp2).
- Session 18 is **Re-check 3** (kind `recheck`), built from the Session 17 page with `check` / `single` switched on as in Session 13:
  one level B story read once (`s18-read-beach`), 16 new words covering every pattern taught (items carry `gpc`; `summary.words.by_pattern`),
  the four made-up words missed in the Session 14 check (`s18-alien`), 18 tricky words (the first 12 are the S8 / S13 set: `summary.tricky_same12`),
  14 spellings with no retry (the first 10 are the S8 / S13 set: `summary.spell_same10`; `before` = her last spelling), a hearing step
  (short u / a under ph1, ee / short i under ph5) and free writing with no frame or buttons. The parent is asked to tap every misread
  word in the story and to count self-corrections, because the S16 audio showed more errors than were marked.
- Audio review: when the transcript and the parent marks disagree, keep the status on the parent marks, add an `AUDIO` evidence note
  to the skill, and check again at the next re-check. If one pass of the model loops on a phrase after a long pause, transcribe the read
  in 30-second pieces.
- Re-check 3 result (S18): word-level skills up (same 12 tricky words 7 → 10 → 11, same 10 spellings 4 → 9 → 9), but story reading flat since S8
  (about 90% on marks, 87% on audio, about 20 wcpm). The audio was below 90% again, so fl2 and sp3 went back to `learning`, ph6 to `not_yet` (ar / or).
- Session 19 (training) is built from the Session 17 page (training engine; S18's re-check parts not needed). Story first, **70 words** (longer, passage text
  23 px so it fits without scrolling), read twice. New: the `pick` step can show a sentence (`sent: true`, items carry `sent` and `say`; more than two options
  wrap two per row) with `feedback: "wword"` (look-alike w-words went / want / what / were / was / saw, tips in `WTIP`; logged under fl1). New writing
  **read back and fix** (`readBack: true`): after Done, the phone reads her text exactly as typed and she can edit it; the item keeps `first_text`,
  `edited_after_reading`, `write_ms`, `fix_ms`, `first_stats`. ar / or re-taught (teach, rule, hear, read with jarn / torb, spell). Cold check of her S18 slips.
  The parent wrap-up reminds about 10 minutes of book reading off the phone each day (the parent agreed to this on 18 Nov).
- Session 19 result (21 Nov): story 'The red hat' read 1 cold 94% on marks (audio 90-94%), 19 wcpm; read 2 after the model read 99%, 46 wcpm.
  Cold pace still about 20 wcpm since S8, but repeated reading more than doubles it. want/went is the core w-word swap (story, pick step, quick words);
  pond -> 'puddle' (guessing by meaning). ar / or taught again and all right the same day (jarn, torb read for the first time): ph6 not_yet -> learning,
  check it cold in S20. Still not holding cold: queen, ou (sound, zoud), hut/hat by ear, boat/seat spelling. Read back and fix: she edits by sound
  (icecrim -> icecreim). Audio transcribed with whisper.cpp (pywhispercpp) and the model in data/models/. progress.json next_session now points to 20.
- Session 20 (training) is built from the Session 19 page (same engine, ids `s20-`). Story first, "The storm" (71 words), read twice; it carries
  the S19 ar / or words (farm, dark, start, barn, storm, short) as a cold check in real text. Cold check: ar / or in new words (shark, horn, star, sport,
  made-up zarp), queen, green, cloud; short u / a by ear (hut, mud, cut, fun); spelling park, corn, boat, seat. **want or went?**: 8 sentence items in the
  `pick` step (`feedback: "wword"`), mostly want / went, with an intro screen (want has an a, went has an e). ou / ow re-taught (teach, rule, hear with
  wrong spellings lowd / cou / rownd / nou / showt, read with made-up voud, spell out, loud, cow). `aiTip` / `teamSays` now say "ow" for cow, now, how, wow.
  Look-cover-write dropped (3/3 in S19) to keep it near 25 min (S19 took 34). Tested end to end at 412×915 in headless Chrome (playwright): no script errors,
  no sideways scroll, story fits without scrolling, result JSON checked.
- Session 20 result (21 Nov, 27 min): story 'The storm' read 1 cold 93% on marks (audio 90-93%), 19 wcpm; read 2 97%, 32 wcpm. ar / or held cold
  (shark, horn, star, sport, zarp 5/5; dark, storm, short in the story; barn missed, start shaky). hut / hat by ear 4/4; boat, seat right first try.
  want or went? 7/8 (when -> went), want read but slow (21 s). ou / ow 14/14 after re-teach, but cloud (cold) and loud (story) not read before it.
  New pattern: past-tense words read or written in another form (sat -> 'said' / 'sit', sang -> 'sing', went -> 'were', "Then it rains").
  Read back and fix: no change ('footboll' kept). No status changes. next_session now points to 21.
- Session 21 (training) is built from the Session 20 page (ids `s21-`). Story first, "The lost ball" (69 words), read twice, full of past-tense
  words (went, had, kicked, looked, found, jumped, sat, saw, sang) and ou / ow (out, round, found, brown). Cold check: cloud, sound, proud, made-up zoud,
  barn, start, corn, saw; cold spelling shout, round, brown, corn. NEW **yesterday words** (past tense, logged under wr15): a rule screen (sit → sat,
  jump → jumped, go → went), a `pick` step with `sent: true` where the phone says the sentence with "beep" for the gap (items carry `say` with beep and
  `full` with the word; the wword feedback speaks `full`, tips in `WTIP` as "Today I sit. Yesterday I sat."), then reading the past forms alone
  (sat, sang, ran, had, jumped, looked, played, kicked; `ed: true`). Read back and fix now asks her to point to each word and find ONE word to fix.
  No phonics teaching block and no hearing step (shorter). Summary keys: `past`, `past_read`. Tested end to end at 412×915 in headless Chrome: no errors,
  no sideways scroll.
- Session 21 result (21 Nov, evening, 31 min): story 'The lost ball' read 1 cold 91% (audio agrees), 17 wcpm; read 2 99%, 49 wcpm (best gain).
  5 of 6 story errors were ou / ow (out -> 'up', round, found, brown, down): ou does not hold in text even the day after re-teaching (cloud missed cold
  3 times). Past tense fixed: sat, sang, kicked, looked, jumped right in the story; yesterday words 8/8 in sentences and 8/8 alone. ar / or cold 3/3.
  Spelling: round -> 'roud', brown -> 'broun'. Read back with "find one word": changed 'prazate' -> 'prezet'. Status: ph11 learning -> secure.
  Session 22 is the last training before Re-check 4 (S23). progress.json next_session now points to 22.
- Session 22 (training, last before Re-check 4) is built from the Session 21 page (ids `s22-`). Story first, "The brown cow" (71 words, full of ou / ow:
  out, brown, cow, loud, found, round, ground, shout, mouse, down), read twice. Cold check cloud, ground, clown, now, torch, march, when. ou or ow? with a new
  rule card (ow before n: down, brown, town; `aiTip` / `teamSays` now cover -own), tap the spelling (6, wrong spellings broun / rownd / doun / clowd / toun /
  fownd), ou / ow **phrases** (`phrase: true`, fl5), spell round, brown, cloud, down; when or went? (6 sentence items, `feedback: "wword"`); quick words.
  Read-back checklist asks for a doing word in each sentence. Summary keys: `hear`, `phrases` (`not_smooth`), `spell`, `wpick`. Tested end to end at 412×915
  in headless Chrome: no errors, no sideways scroll, story fits without scrolling.
- Session 22 result (22 Nov, 39 min active + a 26-min pause between read 1 and the model read): story 'The brown cow' read 1 cold 96% on marks
  (audio 93-96%), 19 wcpm; read 2 100%, 41 wcpm. ou / ow in text much better (only loud missed; S21 missed most). Cold: cloud read at last; clown not.
  "ow before n" (brown, town) not yet: 'broun', 'toun'. when right alone and in sentences, but when -> 'what' in the story; quick words all read but slow
  (what 35 s). Writing: verbs in both sentences (swam, ate) and she fixed 'sooems' and 'watrmelen' herself after read back. No status changes.
  Next: Session 23 = Re-check 4. progress.json next_session now points to 23.
- Session 23 is **Re-check 4** (kind `recheck`), built from the Session 18 page (same steps, `check` / `single` on, ids `s23-`). Story "Under the tree"
  (level B, 54 words, read once, no pattern loaded); 17 new words for every pattern taught (`summary.words.by_pattern`; adds ue, ou, ow); the same four
  made-up words, the same 18 tricky words (`summary.tricky_same12` with s8 / s13 / s18 counts), the same 14 spellings (`before` = her S18 answer,
  `same10` = the first 10) and the same 6 hearing items as S18; free writing about a new picture (kite, wind, tree), no frame. The wrap-up story table
  compares with S18 / S13 / S8. The parent is asked to do it on its own day. Tested end to end at 412×915 in headless Chrome: no errors, no sideways scroll.
- Session 23 result (Re-check 4, 24 Nov, morning, own day, 23 min): cold story 'Under the tree' 98% (under), 22 wcpm (S18 91% / 20, S13 91% / 13,
  S8 95% / 21; audio 94-98%): best accuracy of the re-checks but pace still about 20 wcpm, word by word. Questions 3/3. New words 15/17 (bright, count
  not read). Alien words 4/4 (S18 2/4). Tricky 17/18 (what not read), first 12 11/12. Spelling 12/14: boat -> 'bout' again, seat -> 'saet'.
  By ear ship -> sheep. Free writing: 2 sentences, capitals and full stops, no help. Status: ph6, ph12, co7, sp2 -> secure.
- Session 24 (training) is built from the Session 22 page (ids `s24-`). Story "The boat at night" (69 words: igh night, high, bright, lights; ou out,
  count; what, under, ship, seat, boat, coat) read twice. Cold check bright, count, what, under, clown, boat, seat. igh rule (`RULE`, 3 cards) then igh
  words read by the parent-scored `s24-igh` step (night, light, right, high, bright, fright, sight, alien vight; summary key `igh`). Hear and tap long ee
  vs short i (ship/sheep, seat/sit, feel/fill, bin/bean, heat/hit; `aiTip` now gives a short-i tip). Phrases (fl5), spell boat, seat, night, count;
  what or when? (`wpick`, mostly what); quick words what, come, again, here, with, when. Tested end to end at 412×915 in headless Chrome: no errors,
  no sideways scroll. progress.json next_session now points to 24.
