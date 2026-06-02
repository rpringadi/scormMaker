# scormMaker

A tiny bash script that turns a plain-text study page plus a multiple-choice quiz into a [SCORM 1.2](https://scorm.com/scorm-explained/) `.zip` package, ready to upload to any SCORM-compliant LMS (Moodle, Canvas, Blackboard, TalentLMS, etc.).

No authoring tool. No GUI. No dependencies you don't already have. One bash script, two text files, one zip out.

## Why

Commercial SCORM authoring tools (Articulate Storyline, iSpring, Captivate) are great if you're building polished interactive courses with branching scenarios. They're overkill — and pricey — if all you need is a *theory page + 10-question quiz* repeated across many topics.

scormMaker is built for that case: you have a template you want to stamp out 50 times with different content. Drop your text files in, run the script, get a SCORM zip. Loop over a directory and you've batched a whole curriculum.

## Quick start

```bash
scormMaker "History of Canada" tmp/context.txt tmp/quiz.txt history-of-canada.zip
```

Upload the resulting `history-of-canada.zip` to your LMS as a SCORM activity. Done.

Sample inputs are in [`tmp/`](tmp/) — feel free to copy them as a starting point.

## Requirements

- `bash` 4+
- `python3` 3.6+ (stdlib only — `json`, `re`, `html`. No `pip install`.)
- `zip` (InfoZIP CLI; preinstalled on macOS; `apt install zip` on Debian/Ubuntu)
- `mktemp` (POSIX; always present)

Verify with:

```bash
command -v bash python3 zip mktemp
```

The generated SCORM package itself has zero runtime dependencies beyond a modern browser — it's just static HTML/JS in a zip.

## Input format

### `context.txt`

Plain text. The study material your learner reads *before* taking the quiz. Paragraphs are separated by blank lines. No special markup needed.

```
History of Canada (Student Overview – 1 Page)

The history of Canada spans thousands of years, beginning long before
European contact. The earliest known inhabitants were Indigenous peoples...

European exploration began in the late 15th and early 16th centuries...
```

### `quiz.txt`

Numbered questions, A–D options, one correct answer marked with `(*)`. Blank line between questions.

```
1. Who were the earliest known inhabitants of Canada?
A. Vikings
B. Indigenous peoples (*)
C. Romans
D. Spanish settlers

2. Which country first established large settlements in Canada?
A. Spain
B. France (*)
C. Portugal
D. Germany
```

The `(*)` marker is **stripped from display** but kept inside the package for client-side scoring.

## What gets generated

A standards-compliant SCORM 1.2 zip with:

- `index.html` — the SCO: theory at the top, quiz below, a Submit / Retake / Exit-activity button flow at the bottom. Self-contained, no external assets.
- `imsmanifest.xml` — the SCORM manifest declaring a single SCO with a mastery score of 70.

The bundled JavaScript:

- Renders questions with radio-button options
- Grades client-side on submit
- Reports `cmi.core.score.raw`, `cmi.core.score.min`, `cmi.core.score.max`, and `cmi.core.lesson_status` (`passed` / `failed`) back to the LMS's SCORM 1.2 runtime
- Calls `LMSInitialize`, `LMSCommit`, and `LMSFinish` at the right moments
- Lets the learner retake the quiz within the same attempt (unlimited retries) if they fail

## Pass mark

Pass mark is **70%** by default (in both the JS scoring and the `<adlcp:masteryscore>` in the manifest). Change it by editing the `PASS_MARK=70` line near the top of [`scormMaker`](scormMaker).

## Customising

- **Colors:** the inline `<style>` block in the generated HTML uses Moodle-friendly blue (`#0f6cbf`) for primary buttons. Edit the `cat > "$WORK/index.html"` block to restyle.
- **Exit behavior:** the *Exit activity* button (shown after a passing score) walks up the iframe chain looking for the LMS's SCORM player and triggers its own Exit link if found; falls back to `/my/` (works on any host).
- **Question count:** unlimited. Pass 5 or 50 — the format scales.
- **Languages:** `<html lang="en">` is hard-coded. Edit if you're shipping non-English content.

## Using with Moodle (tested target)

1. Course homepage → *Turn editing on* → *Add an activity or resource* → **SCORM package**
2. Upload your `.zip`, set the activity name
3. Under **Activity completion**, set "Add requirements" → tick *Require view*, *Require passing grade*, and *All SCOs marked completed*
4. *Save and return to course*

For course-level completion to light up the green checkmark on the course card, also enable **Course completion** under the course settings and tick the SCORM activity as a completion criterion.

## License

[MIT](LICENSE). Use it, fork it, ship it.

## Contributing

Issues and PRs welcome. The script is deliberately small (~250 lines) — if you find yourself wanting to add a build system, plugins, or a config file, you might be looking for a different project.
