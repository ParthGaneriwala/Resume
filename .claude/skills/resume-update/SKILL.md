---
name: resume-update
description: Update, revise, or finalize Parth Ganeriwala's CV/resume (the LaTeX awesome-cv project synced between github.com/ParthGaneriwala/Resume and Overleaf). Use this whenever the user wants to edit any section (Education, Skills, Experience, Publications, Program Committees, etc.), sync the CV against LinkedIn or Google Scholar, fix CV layout/pagination bugs, add new publications, or push CV changes to a branch. Always consult this before touching files in this repo, even for a small wording tweak, since it captures known template bugs, sourcing rules, and the git workflow this user expects.
---

# Resume Update

This repo is a XeLaTeX CV (the `awesome-cv` class) kept in sync with an Overleaf project via GitHub. The owner uses it for job applications and immigration filings, so accuracy and consistency matter more than speed.

## Ground rules (non-negotiable)

1. **Never fabricate.** No invented dates, numbers, venues, titles, or bullet content. If a fact isn't already in the document and isn't something the user just told you, don't guess — ask, or leave a visible `\textbf{[CONFIRM: ...]}` placeholder in the PDF (not a LaTeX comment — comments are invisible in the rendered output and won't get noticed).
2. **Verify against primary sources before trusting the existing CV.** The CV has had real errors in it before (wrong venue type, merged/blended metrics from different roles, stale end dates). LinkedIn (ask the user to paste — it's login-gated, `WebFetch` and `claude-in-chrome` may not reach the detail view) and Google Scholar are more authoritative than what's already written down. When the user pastes LinkedIn text, treat it as ground truth over existing CV content, and point out discrepancies rather than silently picking one.
3. **This CV must stay generic** — no mentions of NIW, EB-2, petitions, visas, or other immigration-specific language belong in the document itself. That context, if it comes up in conversation, stays out of the files.
4. **Ask before big structural changes.** Splitting/merging entries, reordering sections, or adding sections the user didn't ask about (even if a source like LinkedIn reveals they'd be more accurate) should be flagged as a question, not done silently. Small, unambiguous corrections (a typo, a date that's provably wrong) don't need a question first — just make the fix and say what you changed.

## Repo layout

- `resume_cv.tex` — main file; `\input{cv-sections/*.tex}` lines control which sections compile and in what order. Commented-out `\input` lines mean a section is drafted but not currently included — check this before assuming a section doesn't exist.
- `cv-sections/*.tex` — one file per section (education, skills, experience, writing [= Publications], committees [= Program Committees], honors, extracurricular, certifications, presentation, formal_experience). Several of these exist but are excluded from the compiled doc; read them before assuming content needs to be created from scratch.
- `awesome-cv.cls` — the template. Has had real bugs (see below) — don't assume template code is correct just because it's boilerplate.
- No `.gitignore`; compiled PDFs/aux files are not tracked historically — don't commit them, just clean them up (`rm -f *.aux *.log *.out *.fls *.fdb_latexmk *.synctex.gz *.pdf`; if `rm` fails with "Device or resource busy" the PDF viewer likely has it open — just skip removing it and `git add` only the source files you intend to commit).

## Workflow

1. **Check for drift before editing.** The GitHub repo, Overleaf, and any zip/export the user hands you can all be out of sync (this has happened — GitHub was behind local edits that never got pushed from Overleaf). Clone/pull fresh and diff against whatever the user gave you (`diff -bB` to ignore line-ending noise — `core.autocrlf=true` means CRLF/LF differences are cosmetic, not real changes) before assuming either source is current.
2. **Work on a feature branch, never main directly.** Naming convention used so far: `cv-update-YYYY-MM`. Reuse the existing open branch for the session rather than creating a new one each time, unless the user asks for a fresh one.
3. **Go section by section, following the user's lead** — don't restructure the whole document unprompted even if you spot other issues. Mention what you noticed, ask if they want it addressed now or later.
4. **Bullet-writing standard** (the user has explicitly asked for this): skill/action-verb + action + quantified result, not task-listing. Concretely:
   - Lead each bullet with a strong, varied action verb — don't reuse the same verb twice within an entry or across adjacent entries.
   - Every bullet should show an outcome/result where the source material supports one (a %, a count, a concrete deliverable) — don't invent metrics that aren't in the source.
   - Before finalizing adjacent entries (e.g., two roles at the same employer), check for duplicated claims — near-identical bullets restated with slightly different framing should be merged into one, not left as two.
   - Cut bullets that are pure tool-listing with no independent result; fold the tool mention into a nearby results-oriented bullet instead.
5. **Recompile after every edit** and check the log, not just the last few lines:
   ```
   xelatex -interaction=nonstopmode resume_cv.tex
   ```
   Grep the log for `Overfull` and `alignment` warnings, not just the tail — a warning early in the log (e.g. in the Skills table) won't show up if you only check the last 20 lines, but it can mean real content is running off the page. If a warning looks new, isolate whether it's actually new or pre-existing with `git stash` + recompile + `git stash pop` before spending time "fixing" something that was already broken and just never got noticed.
6. **Visually inspect the compiled PDF** (`Read` tool supports PDF pages) before calling anything done — LaTeX compiling without errors does not mean the layout is correct. Check page boundaries, especially after adding content to a section whose table/column widths are fixed-width (see known bugs below).
7. **Clean build artifacts, commit only source files**, write a commit message that explains *why* (not just what — matches this user's git log style), push to the branch, and give a short summary of what changed plus what's still open/unconfirmed. Don't merge to main or push to Overleaf without being asked.

## Known template bugs already fixed (don't be surprised if you hit variants of these)

- **`cvskills` table (Skills section) had a fixed-width second column at `0.9\textwidth`**, which combined with the label column already overflowed the page by ~106pt. It was invisible until a long enough skill line was added. Fixed by narrowing to `0.69\textwidth` in `awesome-cv.cls`. If you add a lot of content to the Skills section again and see text running off the right margin, this is the likely cause — check the current fraction in `awesome-cv.cls` (search `cvskills`) before assuming it's something else.
- **`\cvsection` had no page-break awareness**, so a section heading (e.g. "Publications") could get stranded alone at the bottom of a page with all its content pushed to the next page, wasting most of a page. Fixed with `\RequirePackage{needspace}` + `\Needspace*{5\baselineskip}` inside the `\cvsection` macro. If similar orphaning shows up elsewhere (e.g. inside `\cventry` or `\cvsubsection`), the same fix pattern applies.
- **`\cvsubentry`/`\cvsubentries` exist in the class** (for grouping multiple roles under one company header, like LinkedIn's nested display) **but are unused and untested anywhere in the actual content** — the internal `\ifthenelse` logic looked fragile when read closely. Prefer plain repeated `\cventry` blocks (the pattern already used for multi-role employers like Avidyne) over introducing `\cvsubentry` for the first time without testing it in isolation first.

## Source-of-truth hierarchy

When facts conflict, prefer in this order: (1) what the user just told you directly in conversation, (2) LinkedIn (pasted by user), (3) the actual publication record (IEEE Xplore / DOI / journal site), (4) Google Scholar, (5) whatever the CV currently says. The CV itself is the least reliable source — it's the thing being corrected.
