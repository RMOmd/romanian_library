---
name: papermorph
description: Make or edit animated, narrated, interactive web books from reference PDFs using Gemini in Antigravity. Use for book intake and planning, storyboards, chapter SVG animations and quizzes, TTS narration, AI illustration generation, and cover/contents bookshelf pages. Deliver runnable static book files.
---

# Papermorph for Gemini

Papermorph transforms reference PDFs (textbooks, language guides, science/math manuals, literature) into **alive, narrated, interactive web books**.

Technically, each chapter is a static web app consisting of one **1600×900 SVG stage** driven by a timeline player. **Beats** pair spoken audio narration clips with timed vector animations, camera moves, and interactive quizzes.

When running on Gemini in Antigravity, Papermorph leverages Gemini's strong multimodal capabilities, Antigravity's interactive UI modals (`ask_question`), built-in image generation (`generate_image`), browser preview (`browser_subagent`), and automated TTS pipelines (`edge-tts`).

---

## Core Philosophy: What Makes a Chapter Great

1. **Alive**: Elements act and change with weight, easing, and timing rather than sliding like static cards.
2. **Composed**: Each moment is one unified visual composition. Text sits immediately beside what it names.
3. **True**: Motion, labels, phonetics/grammar rules, and calculations agree at every millisecond.
4. **Smooth**: 60fps stutter-free SVG rendering, including on Firefox.
5. **Interactive**: Students learn by doing inside the canvas (tapping words, completing blanks, matching pairs, solving quizzes).

---

## Workspace Layout

```text
books/<book>/
  book.pdf, sections.json      source and page map (private)
  pages/chNN/text.md, *.png    extracted reference material (private)
  style.json                   agreed style every chapter anchors to
  chapters.md                  chapter roadmap and status
  chapters/chNN.md             storyboard notes, errata, feedback
  images/                      inbox, SOURCES.md, processing scripts
  review/chNN/                 storyboard sheets, check output
content/<book>/chNN/           narration.<lang>.json and TTS cache
tools/<book>/                  verification & testing scripts
site/<book>/                   the static book (delivered output)
  index.html, unit-art.js      cover + contents, animated unit sketches
  lib/engine.*, volume.*,      the player engine (copied by scripts/engine.py)
      contents.*, units.*
  lib/book.css                 book engine settings and style overrides
  lib/*.js, img/               book element library, sprites & generated images
  chNN/index.html              chapter lesson application
  chNN/audio/<lang>/           beat MP3s + timings.js
```

`SKILL` refers to this skill's folder: `.agents/skills/papermorph`.
`<book>` is the book slug (e.g. `romanian-easy`), `<lang>` is its primary language code (e.g. `ro`, `ru`, `en`).

---

## Antigravity / Gemini Tool Integration

When operating as Gemini, use the native Antigravity toolset:
- **`ask_question`**: Used in **Intake** and **Review** rounds. Always present 2–4 concrete choices from the book, with the recommended option first (prefixed with `(Recommended)`).
- **`generate_image`**: For illustrated scenes (Tier 2/3), characters, avatars, and thematic backgrounds. Save directly to `site/<book>/img/`.
- **`run_command`**: Execute Python scripts in Windows PowerShell (`python` / `uv run ...`).
- **`browser_subagent`**: Verify playback live in Chromium/Firefox, inspect console logs, and record playback clips.
- **`write_to_file` / `replace_file_content`**: Author narration JSON, chapter HTML, and styles.

---

## Pipeline

Follow this 5-stage pipeline:

### 1. Split & Map
- Locate the reference PDF in the workspace (or ask via `ask_question`).
- Run outline: `uv run --with pymupdf python .agents/skills/papermorph/scripts/outline.py books/<book>/book.pdf --level 1 -o books/<book>/sections.json`.
- If bookmarks are absent, preview initial pages with `python .agents/skills/papermorph/scripts/split_pages.py books/<book>/book.pdf --pages 1-15 --out books/<book>/pages` and create `sections.json`.
- Extract chapter text: `python .agents/skills/papermorph/scripts/split_pages.py books/<book>/book.pdf --sections books/<book>/sections.json --out books/<book>/pages --text-only`.

### 2. Intake (via `ask_question`)
Follow [intake.md](references/intake.md):
1. **Subject & Target**: Greet the user, summarize the book's audience, learning method, and primary languages.
2. **Select Pilot Chapter**: Propose 2–3 representative chapters; recommend one that showcases typical challenges.
3. **Select Tier**:
   - **Tier 1 (Diagrams that change)**: Best for grammar tables, math, linguistic rules, abstract derivations.
   - **Tier 2 (Illustrated scenes & characters)**: Characters speaking in real situations, dialogue scenes, storytelling.
   - **Tier 3 (Visual documentary / Language Immersion)**: High-fidelity illustrations (`generate_image`), cultural immersion, vocabulary art.
4. **Voice & Language**:
   - Romanian: `ro-RO-AlinaNeural` (Female), `ro-RO-EmilNeural` (Male).
   - Russian: `ru-RU-SvetlanaNeural` (Female), `ru-RU-DmitryNeural` (Male).
   - English: `en-US-AndrewMultilingualNeural`, `en-US-AvaMultilingualNeural`.
5. **Write `books/<book>/style.json`**.

### 3. Initialize Book Folder
Run setup as described in [site.md](references/site.md):
- Create directory structure under `site/<book>/` and `content/<book>/`.
- Run engine copy: `python .agents/skills/papermorph/scripts/engine.py site/<book>`.
- Copy templates from `.agents/skills/papermorph/assets/templates/` (`book.html`, `chapter.html`, `unit-art.js`).

### 4. Chapter Authoring Loop
For each chapter:
1. **Analyze source**: Read `pages/chNN/text.md` and relevant page images. Select key linguistic/conceptual points.
2. **Draft Narration & Timing Marks**:
   - Structure beats: `intro` -> `lesson beats` -> `q1` (quick check) -> `lesson beats` -> `wrap` -> `final1` (practice) -> `finish`.
   - Write `content/<book>/chNN/narration.<lang>.json` using `[[mark]]` tags before key trigger words.
   - Synthesize speech and extract timings:
     ```powershell
     uv run --with edge-tts python .agents/skills/papermorph/scripts/tts.py content/<book>/chNN/narration.<lang>.json site/<book>/chNN/audio/<lang>
     ```
   - This writes `<beat>.mp3` and `site/<book>/chNN/audio/<lang>/timings.js`.
3. **Generate Assets (if Tier 2/3)**:
   - Use `generate_image` for character portraits, scenic backdrops, or vocabulary flashcard illustrations.
4. **Author Chapter SVG/JS (`site/<book>/chNN/index.html`)**:
   - Viewport: `1600 × 900`.
   - Follow [engine.md](references/engine.md) and [authoring.md](references/authoring.md).
   - Bind animations to narration marks using `m('mark', offset)`.
   - Clear scenes between beats using `const p = panel(0);`.
   - Add interactive questions via `quiz(BAND, [...], done)`.
5. **Lint & Verify**:
   - Run static checks:
     ```powershell
     python .agents/skills/papermorph/scripts/lint.py site/<book> chNN
     ```
   - Preview with zero-cache server:
     ```powershell
     python .agents/skills/papermorph/scripts/serve.py 8765 site
     ```
   - Use `browser_subagent` to open `http://localhost:8765/<book>/chNN/`, test playback, and verify console is clean.

### 5. Finalize & Register
- Register chapter in `site/<book>/index.html` (`units` list in `Contents.book`).
- Set `CHAPTER.next = '../chNN+1/'` in the previous chapter.
- Update `books/<book>/chapters.md`.

---

## Gemini Implementation Rules & Invariants

When writing code as Gemini:
1. **Never Truncate Output**: Output complete, valid, working JavaScript and SVG code. No placeholders or incomplete closures.
2. **Strict Mark Consistency**: Every `m('markName')` called in JS **must** exist in `narration.<lang>.json` as `[[markName]]`.
3. **Deterministic State**: Replays (seeking to any beat) must reconstruct the scene reliably. Put persistent objects on `S`, and wipe transient visuals with `panel(0)`.
4. **Canvas Discipline (1600×900)**:
   - Header safe area: y: 40–120.
   - Main illustration / animation zone: x: 120–1480, y: 120–640.
   - Question band (`BAND`): y: 686–886. Leave upper scene visible during quizzes!
5. **Language Learning Excellence**:
   - For language books, combine phonetic audio, clear spelling, translation chips (`chip(...)`), stress markers, and interactive blank-fills (`blanks(...)`).
