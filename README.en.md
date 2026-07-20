# form-tailor

[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

[한국어](README.md) · **English**

A [Claude Code](https://claude.com/claude-code) skill that **reproduces *your* document format**: give it a sample and it builds new documents in that exact shape.
It ships no template. The format is always a **runtime input**, and after generating, the skill checks its own output against the sample line by line and reports where it matched and where it didn't.

<!-- demo GIF goes here -->

## Why no bundled templates

There is no shortage of tooling for Korean documents. But most of it solves **"Markdown → hwpx"** — turning *content into a format*, where the format is the one the tool decided on. (HWP/HWPX is the file format of Hangul Word Processor, the de facto standard for Korean institutional paperwork.) What working professionals need more often runs the other way: the target format is already fixed, and it was fixed by **the commissioning body or the department above you**, not by your tool.

Bundling per-institution templates fails twice. **Coverage never arrives** — formats differ across institutions, and differ again by document type *within* an institution. And **they aren't mine to ship**: putting another organization's form files into a public repository is something you simply shouldn't do. So the design goes the other way — **the format is a runtime input, not code** (bring-your-own-template). This repository contains no institutional form assets of any kind, and even the bundled example is a fictional form rather than a real one.

→ [Why I built this · detailed usage](docs/why.md)

## How it works

```
[Sample format] ──parse──▶ [Format profile] ──┐
                                              ├──▶ [New document in that format] ──▶ [Fidelity check]
[New content] ────organize────────────────────┘
```

There are **two inputs**, and both are required — a **format sample** to follow (`.hwp` / `.hwpx` / `.docx`; `.pdf` for structural reference only) and the **content to place in it** (new copy, notes, or a draft already written in some other format). Ask for "our institution's format" without providing a sample and the skill **asks for one rather than inventing a format.**

1. **Profile extraction ★** — section skeleton, per-level bullet markers and indentation, sentence-ending register, title/date/signature conventions and table practice are read out as data, and **the skill stops here for your confirmation.**
2. **Content mapping** — content is placed into the skeleton and the register rules applied. Missing information is not invented, data is not forced into the sample's tables, and figures and proper nouns are untouched.
3. **Generation** — output in the profile's format (kordoc for hwpx, python-docx for docx, explicitly re-applying the formatting that was **observed**). The original sample is never overwritten.
4. **Fidelity check** — the output is compared against the sample profile item by item and reported as a table ([full worked run](examples/example-walkthrough.md)).

## The fidelity check

Output that's been shaped to a format generally *looks* right. The problem is that **looking right and being right are different things** — three items left in the wrong sentence register, a sub-item indented two spaces instead of three, a mandatory "Attachments" section missing entirely: none of that jumps out at the eye. **That comparison, more than the generation, is what this skill is for.**

✅ matches the sample · ⚠️ a partial match, or **a decision the skill deliberately left to you** (an ambiguous register conversion, a placeholder where information wasn't available, a table-column conflict) — **this is the column a human needs to read**, and answering applies it directly · 🔴 missing or in violation. If even one line is 🔴, the skill does not report the document as format-compliant.

## Install

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92/form-tailor.git
```

Restart Claude Code and it will pick up relevant requests automatically. The skill's trigger description is written in Korean, so Korean phrasing invokes it most reliably — you can always name the skill directly instead. Its output follows your sample.

## Usage

```
Format this week's items exactly like the attached weekly report   ← reuse last time's shape
Lay this draft out according to the attached form template         ← fill an empty form
Rebuild this Markdown draft in the attached institutional format   ← port a draft over
Sub-items are indented two spaces, not three                       ← ★at the profile check
Convert the 3 items still in declarative form                      ← when the check returns ⚠️
```

It stops once, at the profile check. Proceeding on a misread profile means rebuilding the whole document; fixing one line there is far cheaper.

## Scope & limits

- `.hwp`/`.hwpx` parsing, form-field filling and structural comparison need the [kordoc](https://github.com/chrisryugj/kordoc) MCP server. Without it the skill falls back to python-docx / LibreOffice, but **hwpx format preservation is limited**
- **It neither produces content nor invents formats.** Research and writing are not this skill's job, and figures and claims aren't changed — only the format is transplanted. Real names and contact details in the sample's signature block are not carried over either; the checklist notes it
- **It doesn't guarantee compliance with any institution's regulations.** It reproduces the formatting **observed** in the sample you provided
- **A table full of ✅ does not mean every aspect of the format is perfect.** python-docx doesn't resolve style inheritance, so body font and line spacing frequently come back `UNOBSERVED`, and formatting that was never observed can't be checked either. `.pdf` samples are structural reference only
- The verification record published in this repo is a single adversarial smoke test that completed all four phases against a synthetic form ([CHANGELOG.md](CHANGELOG.md))
- Details: [SKILL.md](SKILL.md) · [profile schema](references/profile-schema.md) · [Korean form conventions](references/korean-form-conventions.md) · [fidelity checklist](references/fidelity-checklist.md)

## Series

**Agent-team kits** — [policy-research-kit](https://github.com/parkjui92/policy-research-kit) (policy research reports) · [rnd-proposal-kit](https://github.com/parkjui92/rnd-proposal-kit) (Korean government R&D proposals) · [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit) (social science papers)

**Authoring kit** — [lecture-deck-kit](https://github.com/parkjui92/lecture-deck-kit) (HTML lecture decks with in-browser live editing)

**Standalone skills** — **form-tailor** (this repo — institutional document formats) · [fact-verify](https://github.com/parkjui92/fact-verify) (source verification) · [paper-proofread](https://github.com/parkjui92/paper-proofread) (Korean academic proofreading) · [report-to-brief](https://github.com/parkjui92/report-to-brief) (report compression)

## License

[MIT](LICENSE). No proprietary institutional forms or template files are included (bring-your-own-template principle).
