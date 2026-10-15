---
trigger: model_decision
description: Instructions for building and editing interactive animated books with Papermorph
---

# Papermorph Interactive Book Guidelines for Gemini

When creating or modifying animated interactive books:

1. **Skill Reference**:
   Follow instructions in `.agents/skills/papermorph/SKILL.md` and related documents in `.agents/skills/papermorph/references/`.

2. **Interactive Intake & Review**:
   - Always use the `ask_question` tool when consulting the user on book selection, tier, style, pilot chapter, or changes.
   - List the (Recommended) choice first.

3. **Narration & TTS**:
   - Write `content/<book>/chNN/narration.<lang>.json` with clear `[[mark]]` tags before the trigger words.
   - Generate audio & timings using:
     `uv run --with edge-tts python .agents/skills/papermorph/scripts/tts.py content/<book>/chNN/narration.<lang>.json site/<book>/chNN/audio/<lang>`
   - Spoken Romanian: `"voice": "ro-RO-AlinaNeural"` or `"ro-RO-EmilNeural"`.
   - Spoken Russian: `"voice": "ru-RU-SvetlanaNeural"` or `"ru-RU-DmitryNeural"`.

4. **Visuals & Art Generation**:
   - For illustrated scenes, characters, or cultural backgrounds (Tier 2/3), use the `generate_image` tool to create assets directly into `site/<book>/img/`.

5. **Authoring Quality Invariants**:
   - Stage is strictly 1600×900 SVG.
   - Every `m('mark')` in JavaScript must correspond to a `[[mark]]` in the narration JSON.
   - Clean transitions between beats: use `const p = panel(0);` at the start of a beat.
   - Keep quiz cards (`BAND`) clear of upper canvas illustrations.
   - Always run static linting before delivery:
     `python .agents/skills/papermorph/scripts/lint.py site/<book> chNN`
