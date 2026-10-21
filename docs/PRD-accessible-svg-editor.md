# PRD: Accessible SVG Drawing Studio for Blind Learners

**Working title:** SVGnator (name from the teacher's whiteboard mockup; earlier placeholder: SVG Studio)
**Document status:** Draft v0.2. Updated after building and testing prototype v0.2.1; see section 15 for the changes from v0.1
**Target platform:** Claude.ai published web Artifact (single self-contained HTML page)
**Date:** October 2026
**Prototype:** https://claude.ai/artifact/4M7ymTaozDbSuWBkjZyEqD (v0.2.1)

---

## 1. Problem statement

A teacher is teaching blind students to create graphics by writing SVG code by hand. Today the students write code with no reliable, immediate way to know what their code actually draws. Feedback depends on a sighted person looking at the output, which is slow, breaks independence, and makes iteration (the core of learning to draw with code) very expensive.

The product is a compact, screen-reader-first web editor where a student writes SVG, a live preview renders it (for sighted helpers and low-vision users), and the app produces a spoken, honest description of what the image looks like ("this looks like a sun: a circle with eight lines radiating outward") at a level of detail that matches the image's complexity.

## 2. Goals and non-goals

### Goals
1. A blind student can write, edit, run, and understand an SVG drawing entirely by keyboard and screen reader, without sighted help.
2. The student gets fast, trustworthy feedback on what the drawing looks like, both as a whole ("looks like an owl") and structurally (which shapes are where).
3. The teacher can use the same tool in class and see what the student sees.
4. Students can save their work and come back to it.
5. A student can describe a picture in words and get a small, readable, commented SVG example to study and change. It is a worked example, not a replacement for learning to write SVG.

### Non-goals (v1)
- A drag-and-drop or mouse-based visual drawing tool. The code is the drawing surface.
- Full SVG spec coverage (filters, animation, scripting). See section 7.4.
- Grading, LMS integration, or a full classroom management system.
- Producing tactile graphics hardware output directly (exporting a file that could be embossed is a possible later feature, see section 10).

## 3. Users and personas

**P1: Blind student (primary).** Uses a screen reader (assumed: NVDA or JAWS on Windows, VoiceOver on macOS/iOS; to be confirmed). May use a refreshable braille display. Comfortable with a keyboard. Learning coordinates, shapes, and SVG syntax. Age and prior coding experience unknown (open question).

**P2: Teacher (secondary).** Sighted (assumed). Prepares exercises, watches the preview, helps debug, wants to check whether the AI description is accurate.

**P3: Low-vision student (secondary, assumed).** Uses the visual preview with high zoom or high contrast plus partial screen-reader support.

## 4. Core user stories

| ID | As a... | I want to... | So that... |
|---|---|---|---|
| US-1 | blind student | type SVG code in an editor my screen reader reads correctly | I can write and review my code line by line |
| US-2 | blind student | press one shortcut to hear what my drawing looks like | I know if my code draws what I intended |
| US-3 | blind student | hear a list of the shapes, their positions and sizes in plain words | I can find which line made which part of the drawing |
| US-4 | blind student | be told immediately and clearly when my code has an error, and on which line | I can fix it without guessing |
| US-5 | blind student | choose a short or detailed description | I get a quick check or a full explanation |
| US-6 | blind student | save drawings and reopen them later | I can work across sessions |
| US-7 | blind student | start from a template (empty canvas, sun, face) | I don't start from a blank page every time |
| US-8 | blind student | ask a follow-up question about my drawing ("is the eye inside the head?") | I can verify specific details |
| US-9 | teacher | see the live preview next to the code | I can follow what the student is doing |
| US-10 | teacher | compare the AI description with the actual image | I can tell students when the description is wrong |
| US-11 | blind student | describe in words a picture I want and get simple, commented SVG code for it | I have a worked example to read line by line and change |
| US-12 | blind student | review generated code before it touches my drawing, then replace my code, add the shapes to my drawing, or discard it, and undo that choice | I stay in control of my own code |
| US-13 | blind student | be told whether generated code has errors or warnings before I use it | I never import broken code without knowing |
| US-14 | teacher, or anyone without a screen reader | switch descriptions to the app's own voice right next to the Describe button | I can hear descriptions without opening Settings (proposed, not built) |
| US-15 | teacher | turn code generation off for some lessons | students practise writing code themselves when that is the goal (open question Q9, not built) |

## 5. Functional requirements

Priority: **M** = must (v1), **S** = should, **C** = could (later).

### 5.1 Code editor

| ID | Requirement | Priority |
|---|---|---|
| ED-1 | The editor is a native `<textarea>` (or an editor library proven to work with NVDA, JAWS, and VoiceOver). Rationale: native textareas are the most predictable with screen readers and braille displays; rich code editors vary in support. **Decision needs testing with real users before choosing a library.** | M |
| ED-2 | Accessible label, and a keyboard shortcut to jump to the editor from anywhere. | M |
| ED-3 | Tab key inserts indentation only if the user turns that on; by default Tab moves focus (to avoid a keyboard trap, WCAG 2.1.2). Escape then Tab must always leave the editor. | M |
| ED-4 | "Read current line" and "go to line N" commands. | S |
| ED-5 | Snippet insertion via a menu or command palette (circle, rect, line, ellipse, polygon, path, group, text) with sensible default attributes. | S |
| ED-6 | Syntax highlighting for the visual preview of the code (decorative only, must not change what the screen reader announces). | C |
| ED-7 | Undo/redo relies on native browser behaviour where possible. | M |
| ED-8 | The visual line-number column is decorative and hidden from screen readers. Its height is set by the editor text area, never by the number of lines, and it scrolls in step with the code after typing, scrolling, "go to line", selecting a shape, and resizing. Rationale: in v0.2 long code made the numbers stretch the frame so they no longer matched the code (fixed in v0.2.1). | M |

**Status in prototype v0.2.1:** ED-1 (native textarea), ED-2, ED-3, ED-4, ED-5, ED-7 and ED-8 are built. ED-6 is not. Programmatic edits (snippets, templates, opening a project, generated code) go through the browser's editing command so Control Z still works; a snippet inserted with the cursor below `</svg>` goes just before `</svg>`. Nothing has been tested with NVDA, JAWS or VoiceOver yet.

### 5.2 Rendering and validation

| ID | Requirement | Priority |
|---|---|---|
| RE-1 | The SVG is parsed with the browser's XML parser (`DOMParser`, `image/svg+xml`) when the user runs it (shortcut, button) and optionally on a pause in typing (debounced, user setting). | M |
| RE-2 | Parse errors are announced through an `aria-live` region with line number and a plain-language explanation (e.g. "Line 4: the circle tag is not closed"). Raw parser messages are kept available on demand. | M |
| RE-3 | Before rendering, the SVG is sanitized: remove `<script>`, `<foreignObject>`, event-handler attributes (`on*`), and external references (`href` to remote URLs). Rationale: SVG can execute script; shared drawings must be safe. | M |
| RE-4 | Rendering errors that are not parse errors (e.g. a shape outside the viewBox, zero-size shape, missing `fill` and `stroke` making it invisible) produce warnings, announced the same way. | S |
| RE-5 | The canvas size / viewBox is shown and announced (e.g. "canvas is 200 by 200, origin top-left, y goes down"). | M |
| RE-6 | If `xmlns` is missing (and `xmlns:xlink` when `xlink:` is used), add it automatically on the same line so line numbers stay correct. Students forget it, and without it nothing renders. | M |
| RE-7 | A built-in checker runs before the browser parser and explains common mistakes in plain words with line and column: unclosed tags, missing or unquoted attribute values, no space between attributes, duplicate attributes, `&` not written as `&amp;`, `--` inside a comment, text outside `<svg>`. Rationale: browser parser messages are cryptic and differ between browsers. The checker also records each tag's line so every drawn shape can be traced to its line (SD-1, SD-3). | M |

**Status and decisions in v0.2.1:** RE-1 to RE-7 are built. Run only announces the canvas when it changes. Optional checking on a pause in typing updates the preview without any announcement.
- **RE-3 detail:** the sanitizer also removes `<style>` elements, because the preview is drawn inside the page and a student's CSS would restyle the app itself. It also removes animation elements (`animate`, `set` and similar), because they can change attributes such as `href` after sanitizing. Each removal is listed under Problems as "Removed for safety". This narrows section 7.4's "anything else renders" rule.
- **RE-4 warnings built:**
  - shape with no fill and no stroke
  - line with no stroke
  - white on the white background
  - zero size
  - partly or fully outside the canvas, naming the edge where it is cut off
  - circle using `x`/`y` instead of `cx`/`cy`
  - missing `r`, `width`/`height`, `points` or `d`
  - wrong capitalization (`Circle`, `viewbox`)

  The teacher's mockup itself contained the `<circle x="10" y="10">` mistake, which shows how valuable these warnings are.

### 5.3 Structural description (deterministic, no AI)

This is a computed, reliable description built from the SVG itself. It complements the AI description, which can be wrong.

| ID | Requirement | Priority |
|---|---|---|
| SD-1 | List every visible element with: type, the source line it comes from, colour, position in words (e.g. "top-left area", "center"), and size relative to the canvas. Uses `getBBox()` on the rendered element. | M |
| SD-2 | Relationship hints: "circle 2 is inside circle 1", "line 3 touches rectangle 1", "these two shapes overlap". | S |
| SD-3 | Each list item links to its source line: activating it moves the editor cursor to that line. | S |
| SD-4 | Warnings for things that are likely mistakes: element outside the canvas, fully hidden behind another shape, same colour as background. | S |
| SD-5 | "Where is the point (x, y)?" query: announces which shapes cover that coordinate. | C |

**Status in v0.2.1:** SD-1, SD-2 and SD-3 are built. Each entry reads like "Circle, line 2: gold fill with black outline, in the center, very large (center 100, 100, radius 80)." Colours use the student's own colour word when they wrote one, otherwise the nearest plain name. "Inside" and "overlaps" are computed from the rectangle around each shape, so they are approximate: for example, shapes at the bottom of a big circle can be reported as inside it. SD-4 is partly built (outside the canvas, white on white); "hidden behind another shape" is not. SD-5 is not built.

### 5.4 AI visual description

| ID | Requirement | Priority |
|---|---|---|
| AI-1 | On request (shortcut / button), the rendered SVG is rasterized to PNG in the browser (canvas) and sent as an image to Claude through the Artifact `sample` capability with images enabled. The model describes what it **sees**, not what the code says. | M |
| AI-2 | **Anti-bias rule:** by default, comments, `id`, `class`, and `<title>`/`<desc>` content are not sent, and the code is not sent at all for the "what does it look like" question. Rationale: if the code says `id="owl"`, the model will be tempted to say "owl" regardless of the drawing. The student needs honest feedback. | M |
| AI-3 | Description levels: **Short** (one sentence, "looks like..."), **Detailed** (composition, shapes, colours, positions), selected by the user. Length should also scale with image complexity. | M |
| AI-4 | The prompt instructs the model to state uncertainty ("it could be a sun or a flower"), to say when it does not look like any recognizable object, and not to flatter. | M |
| AI-5 | Optional "intent check": the student types what they meant to draw; the model reports whether the image matches and what differs. Sent **after** the blind description, so the first judgement is unbiased. | S |
| AI-6 | Follow-up questions about the current image (US-8), with the image re-sent each time (the model has no memory between calls). | S |
| AI-7 | Descriptions are not generated on every keystroke (cost, rate limits, and spoken noise). Only on explicit request, or optionally on a long pause. | M |
| AI-8 | If `sample` is unavailable (`null`), not granted, or rate limited, the UI says so clearly and the structural description (5.3) still works. | M |
| AI-9 | The UI makes clear the AI description can be wrong. | M |

**Status and decisions in v0.2.1:** AI-1 to AI-9 are built.
- **AI-2 is satisfied by design:** only a PNG of the drawing on a white background (up to 768 pixels on its longest side) is sent, never the code.
- **Model tiers:** Short uses the "quick" tier for speed and Detailed uses "default". Whether "quick" is accurate enough is unproven (Q10). The platform chooses the actual model and may substitute a tier; the app does not yet show which tier answered.
- **Caching:** identical describe and compare requests within 5 minutes reuse the stored answer, which saves cost and rate limit. Follow-up questions are never cached.
- **Fallbacks:** when images or `sample` are unavailable, the AI controls stay focusable but use `aria-disabled` and explain why when pressed.
- **Safari:** whether it allows drawing the SVG to a canvas for this is unverified (Q11).

### 5.5 Speech output

| ID | Requirement | Priority |
|---|---|---|
| SP-1 | Two modes, user-selectable: (a) **Screen reader mode** (default): descriptions are put in an `aria-live` region and the user's own screen reader reads them; (b) **Built-in voice mode**: the app speaks using the browser's Web Speech API (`speechSynthesis`). | M |
| SP-2 | The app must not speak with both at the same time (double speech is a known usability problem). Built-in voice mode should mark announcements so they are not also pushed to the live region. | M |
| SP-3 | Controls: stop speaking (shortcut), repeat last description, speech rate, voice selection. | M (stop, repeat), S (rate, voice) |
| SP-4 | Every spoken output is also available as text in the page so it can be read with a braille display and reviewed later. | M |

**Status in v0.2.1:** SP-1 to SP-4 are built, and every message, not only descriptions, goes through the same speech path.
- **Live regions:** they exist from page load and are cleared before each message so repeats are announced.
- **App voice:** speaks sentence by sentence, because Chrome stops long utterances after about 15 seconds.
- **Stop:** silences the app's voice and cancels an AI request, but it cannot silence the user's screen reader; the Help dialog says to press Control for that.
- **Known gap:** the app's voice is only reachable in Settings, so someone without a screen reader hears nothing by default. US-14 proposes a visible switch next to Describe. A page cannot reliably detect a screen reader, so it cannot switch modes automatically.

### 5.6 Visual preview

| ID | Requirement | Priority |
|---|---|---|
| PV-1 | Live preview panel of the sanitized SVG, marked so screen readers skip the graphic itself (`aria-hidden` on the preview, the description is the accessible version). | M |
| PV-2 | Optional grid overlay with coordinates, for teacher and low-vision users. | S |
| PV-3 | Zoom and high-contrast background toggle. | S |
| PV-4 | Highlight the element on the current editor line (for the sighted teacher following along). | C |

**Status in v0.2.1:** PV-1 to PV-3 are built: the preview is hidden from screen readers, with a grid with coordinate labels, zoom from 100 to 300%, and a white or black background. PV-4 is not built.

### 5.7 Projects and persistence

| ID | Requirement | Priority |
|---|---|---|
| PR-1 | Save, rename, open, delete drawings. Stored per user with the Artifact `db` capability under the user's private path (`data/users/<id>/`). | M |
| PR-2 | Unsaved-draft protection: autosave the current draft to `localStorage` (wrapped in try/catch) so a reload does not lose work. | M |
| PR-3 | Export: download the `.svg` file (Artifact `downloads` capability) and copy code to clipboard. | S |
| PR-4 | Import: paste or open an `.svg` file. | S |
| PR-5 | Teacher sharing: student shares a drawing with the teacher. Mechanism depends on open question Q5. | C |

**Status in v0.2.1:** PR-1 to PR-4 are built. Each drawing is one private document at `data/users/<user id>/<drawing id>` holding `{name, code, updatedAt}`. The draft and settings are kept in `localStorage`. Delete asks for a second press instead of a browser confirm box. Download uses the `downloads` capability, and Copy falls back to selecting all the code if the clipboard is blocked. PR-5 is not built.

### 5.8 Templates and learning aids

| ID | Requirement | Priority |
|---|---|---|
| TL-1 | Starter templates: empty canvas, and a few simple examples (sun, face, house). | S |
| TL-2 | Short accessible reference for each supported element and attribute, reachable from the editor. | S |

**Status in v0.2.1:** TL-1 is built with four templates: empty canvas, sun, smiling face (matching the mockup) and house. TL-2 is not built.

### 5.9 Create code from a description (new in v0.2)

| ID | Requirement | Priority |
|---|---|---|
| GEN-1 | The student types a description (up to 500 characters) and chooses a detail level: Simple (up to 6 shapes) or More detailed (up to 15). The request goes through `sample` as text only, with no image, so it also works where images are unavailable. | S |
| GEN-2 | The result appears in a separate, read-only "Suggested code" area, never directly in the editor (US-12). | M (if GEN is built) |
| GEN-3 | The prompt limits output to readable beginner code. It uses only `g`, `rect`, `circle`, `ellipse`, `line`, `polyline`, `polygon`, `path` and `text`, with no style, class, id, gradient, filter, transform, script or link. Other rules: the same viewBox as the student's current canvas, colour names, whole numbers, everything inside the canvas, background first, one element per line with two-space indentation, and paths only when needed, using M, L, Q, C and Z. | M (if GEN is built) |
| GEN-4 | Optional plain-language comment before each part (on by default), for example `<!-- Wheels: two black circles under the body -->`. Comments must not contain `--`. | S |
| GEN-5 | Before offering the code, the app checks it with the same checker as the editor (RE-7, RE-4 source warnings) and announces shapes, lines, and errors or warnings (US-13). If it has an error, only "Replace" is offered, so the student can fix it in the editor. | M (if GEN is built) |
| GEN-6 | Actions: Replace my code, Add to my drawing (inserted just before `</svg>`, announcing which lines), Discard. Replace and Add are undoable with Control Z. | M (if GEN is built) |
| GEN-7 | Requests are never cached, because asking again should give a new attempt. Stop cancels a request. Alt+Shift+C jumps to the description field, and Control Enter in that field creates the code. | S |
| GEN-8 | Teachers may need to switch this feature off for some lessons (US-15, Q9). Not built, because a switch in Settings would not stop students turning it back on. | Open |

**Status in v0.2.1:** GEN-1 to GEN-7 are built and tested with a simulated Claude reply. Code quality from the real model is not yet measured (Q12). Generated comments name the subject of the picture, but this does not affect AI-2, because descriptions only ever see pixels.

## 6. Accessibility requirements (non-negotiable)

1. Conform to **WCAG 2.2 Level AA** as the baseline.
2. Fully keyboard operable, visible focus indicator, no keyboard traps.
3. Keyboard shortcuts must avoid the keys screen readers reserve (NVDA/JAWS modifier keys such as Insert and Caps Lock, VoiceOver's Control+Option) and must not depend on browse-mode single-letter navigation. Every shortcut also has a visible button equivalent and the shortcut list is announced by a help command.
   - **v0.2.1 shortcuts:** Control (Command on a Mac) + Enter runs the code. Everything else is Alt (Option) + Shift + a letter: E editor, D describe, L shapes, P problems, R repeat, X stop, S save, K read line, G go to line, C create from a description, H help. Keys are matched by physical key, because Option changes the typed character on a Mac.
   - **Avoided:** T and I, which Chrome uses with Alt+Shift on Windows.
   - **Untested:** conflicts on the students' real setups (Q13).
4. Landmarks and headings so the page is navigable: Editor, Description, Structure, Preview, Projects.
5. Live regions used sparingly: `polite` for descriptions, `assertive` only for errors the user just caused.
6. Compact layout: the user should reach the three core actions (edit, run, describe) within a few keystrokes. Avoid long navigation chains.
7. **Testing with at least one blind user and the teacher is required before v1 is called done.** Automated checkers do not catch most screen reader usability problems. Visual testing by a sighted person is also needed: the v0.2 line-number bug was invisible to screen readers but confusing for the teacher.
8. Controls that cannot work right now (for example AI features without permission) use `aria-disabled`, not `disabled`, so they stay reachable and explain why when pressed.
9. Moving focus is reserved for actions the user started that open or close something (dialogs, "go to line", choosing a shape). Results of Run, Describe and Create are announced, not focused.

## 7. Non-functional requirements

### 7.1 Platform constraints (from the Claude.ai Artifact runtime, contract 0.2.74 as of v0.2.1)
- Single self-contained HTML file, under 16 MB.
- External scripts only from cdnjs.cloudflare.com, cdn.jsdelivr.net, cdn.tailwindcss.com, code.jquery.com; fonts only from Google Fonts. No other network requests.
- AI calls go through the `sample` capability (not a direct API call). First use asks the viewer for consent; calls can be rate limited.
- Storage via `db` (server-side, per user) and `localStorage` (per browser, convenience only).
- Capabilities may resolve to `null`; every feature that depends on one must degrade gracefully.
- Capabilities declared by the prototype: `sample` with images, `db`, `user` (for the private per-user path) and `downloads`. Text-only `sample` calls work even where images are not allowed.
- Each AI request uses the viewer's own Claude usage, and the first one asks the viewer for permission.
- The browser's `alert`/`confirm`/`prompt` boxes are not used. Dialogs are native `<dialog>` elements.

### 7.2 Performance
- Render after "run": under 200 ms for typical classroom drawings (target, to be validated).
- Structural description: under 300 ms.
- AI description: depends on the service; show and announce a "describing..." state, allow cancel.

### 7.3 Privacy
- Drawings are private to the student by default.
- The images are sent to Claude for description; the UI should say so once, in plain words.
- Student age matters: if students are minors, data collection should be minimal (no names in drawings, no profile data stored). Needs confirmation (Q1).

### 7.4 Supported SVG subset (v1)
`svg`, `g`, `rect`, `circle`, `ellipse`, `line`, `polyline`, `polygon`, `path`, `text`, basic presentation attributes (`fill`, `stroke`, `stroke-width`, `opacity`, `transform`). Anything else renders if the browser supports it, but the structural description may only say "unsupported element on line N".

## 8. Layout (updated in v0.2 from the teacher's mockup)

The v0.1 layout put a toolbar at the top. The teacher's mockup puts each action next to what it acts on, and that turned out better for keyboard users: one Tab from the editor reaches Run. v0.2 follows the mockup. Page order:

1. **Header:** app name, current drawing name or "Unsaved draft", Projects, Settings, Help.
2. **Editor column (left on wide screens):**
   - the code editor
   - Run, Save and Open saved
   - Insert shape and Start from (template)
   - Read current line and Go to line
   - Problems (errors, warnings, removed-for-safety notes, each with "Go to line N")
   - Create from a description
3. **Preview (right):** visual only and hidden from screen readers, with the canvas sentence, grid, zoom and background controls.
4. **Description (right, below the preview, as in the mockup):** Short/Detailed, Describe, Stop, Repeat, the latest description, the AI disclaimer, Check what I meant, Ask about the drawing, and Message history.
5. **Shapes (right, last).**

On narrow screens everything stacks in that order. The mockup places the description after the preview, while v0.1 put the feedback first. This does not matter for screen readers, because the preview is hidden from them, so they hear editor, problems, description, shapes.

### 8.1 Mockup review (v0.2)

The whiteboard mockup and the PRD were compared before building:
- The mockup's "Validate" became **Run**: check and draw, with no AI.
- The mockup shows a description but no Describe button, which implied the description runs automatically. That conflicts with AI-7, so Describe stays a separate, explicit action.
- The mockup had no place for RE-2 errors, SD-1 shapes, AI-3 levels, SP-3 Stop or AI-9; all were added.
- The mockup's sample code drew the wrong thing (`x`/`y` on a circle, no viewBox), which became a test case for RE-4.

## 9. Success metrics

- A blind student completes a starter exercise (e.g. draw a sun) without sighted help.
- Students report the descriptions as accurate in at least most cases (measured by teacher spot checks against the preview).
- Time from "I changed my code" to "I know what it looks like" is a few seconds instead of waiting for a sighted person.
- Zero critical accessibility blockers in testing with real screen-reader users.

These need baselines from the teacher's current process; the numbers above are not targets yet.

## 10. Future ideas (not in scope for v1)

- Sonification: hear the shape outline as sound, pitch for height and stereo pan for horizontal position.
- Tactile export: a high-contrast, simplified version suitable for swell paper or embossers.
- Teacher dashboard with assignments and student gallery.
- Step-by-step guided lessons.
- Multi-language UI and descriptions.

## 11. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| AI description is wrong or overconfident | Student learns wrong lessons | Structural description as a reliable second source, uncertainty in prompt, AI-2 anti-bias rule, teacher spot checks |
| Code editor library breaks with screen readers | Product unusable | Default to native textarea; test with NVDA, JAWS, VoiceOver |
| Double speech (app voice + screen reader) | Confusing, unusable | SP-1 / SP-2 mode separation |
| `sample` rate limits during a class | Feedback stops mid-lesson | On-demand only, caching identical images, structural fallback |
| Malicious or broken SVG | Security issue or crash | Sanitization (RE-3) |
| Shortcut conflicts with screen readers | Features unreachable | Section 6 rule 3, configurable shortcuts |
| Code generation replaces learning | Students stop learning to write SVG | Suggested code is shown separately first, comments explain each part, small shape limits, possible teacher switch (GEN-8, Q9) |
| Generated code is broken or does not follow the rules | Confusing errors in the student's drawing | Checked before it is offered (GEN-5); Add is disabled on errors; undo always works |
| Visual-only bugs go unnoticed in screen reader testing | The teacher sees a broken or misleading screen | Sighted visual checks and screenshots in testing (section 6 rule 7) |
| "Inside/overlaps" computed from bounding boxes | Wrong relationship hints for round or diagonal shapes | Labelled as approximate in the UI; exact geometry is a later improvement |

## 12. Assumptions (please confirm or correct)

- A1. Students are fully blind (not only low vision) and use a screen reader.
- A2. They are on desktop/laptop with a physical keyboard, not mainly phones.
- A3. The UI language is English.
- A4. Each student has their own Claude.ai access to open the artifact (needed for `db` storage and `sample`).
- A5. The teacher is sighted.

## 13. Open questions

- Q1. How old are the students, and how much coding experience do they have?
- Q2. Which screen readers, browsers, and operating systems do they use? Any braille displays?
- Q3. What language should the interface and the spoken descriptions be in?
- Q4. Do all students have Claude.ai accounts? If not, the artifact capabilities (AI, storage) will not work for them, and this changes the platform choice.
- Q5. Does the teacher need to see students' saved work, or is sharing code by copy/paste enough for v1?
- Q6. How do students currently check their drawings, and what is the most frustrating part? (Defines the success baseline.)
- Q7. Should descriptions ever mention the code (e.g. "the circle on line 3 is the head") or only the image?
- Q8. Is the teacher willing to test early versions with one or two students?
- Q9. Should code generation always be available, or should the teacher control it per lesson? A real lock needs teacher and student roles, which v1 does not have.
- Q10. Is the "quick" model tier accurate enough for Short descriptions, or should both levels use "default" despite the slower answers?
- Q11. Does Safari allow the drawing to be turned into an image for Claude? If not, AI descriptions need another approach there.
- Q12. How well does the real model follow the code generation rules (shape limit, inside the canvas, simple shapes)? Needs a set of test prompts.
- Q13. Do the Alt+Shift shortcuts conflict with anything on the students' actual screen readers, browsers and keyboards?
- Q14. Should the visible "read aloud with" switch (US-14) be added next to Describe?

## 14. Backlog after v0.2.1 (not yet built)

In rough priority order:
1. Test with at least one blind student and the teacher (section 6 rule 7).
2. Visible speech mode switch next to Describe (US-14).
3. Warning when a shape is hidden behind another (SD-4).
4. Show which model tier answered.
5. Check rasterization in Safari.
6. Measure the quality of generated code.
7. Teacher sharing (PR-5) and a teacher switch for code generation (GEN-8).
8. Element reference (TL-2), editor-line highlight in the preview (PV-4), "where is the point" (SD-5), configurable shortcuts.

## 15. Changes from v0.1

- Working title set to SVGnator, the name from the mockup.
- New goal 5, user stories US-11 to US-15, and section 5.9 (create code from a description).
- New requirements ED-8, RE-6 and RE-7.
- Status and decisions recorded under each requirement section.
- Section 8 rewritten to follow the mockup, with a new 8.1 mockup review.
- Section 6 extended with the shortcut list and rules 8 and 9.
- Section 7.1 updated to contract 0.2.74 and the declared capabilities.
- New risks, open questions Q9 to Q14, and a backlog (section 14).
