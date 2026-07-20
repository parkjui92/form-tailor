# form-tailor

[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

[한국어](README.md) · **English**

> A [Claude Code](https://claude.com/claude-code) skill that **reproduces *your* document format**: give it a sample and it builds new documents in that exact shape.
> It ships no template. The format is always a **runtime input**, and after generating, the skill checks its own output against the sample line by line and reports where it matched and where it didn't.

---

## Why I built this

In Korean institutional work, the real cost of a document is **not the content — it's the format.**

The content is usually already in hand. The person writing the report knows what happened, what's next, and what the problem is. The time goes somewhere else:

- One document uses `□` for section headings and `ㅇ` for items; another starts at `Ⅰ.`. Sub-items are indented one space in one house style and three in another.
- One document is written in *gaejosik*; another is in polite declarative form. Within the same institution, an internal report and an outgoing official letter follow different conventions.
- Some places write dates as `'26. 5. 7.(목)`; others don't. Some put the signature as "title + name" below the heading on the right; others place it differently.
- Tables have to match the previous document from column structure down to header cell shading.

And getting these wrong **gets the document sent back** — no matter how correct the content is. So the actual workflow is: open last time's file next to the new one, and push new content in while eyeballing the format.

> **Two terms, if you're not working in Korean.** **HWP/HWPX** is the file format of Hangul Word Processor, the de facto standard for Korean public-sector and institutional paperwork — roughly what `.docx` is elsewhere, but with its own conventions and its own tooling gap. ***Gaejosik*** (개조식) is the terse outline register of Korean official reports: each item is a clipped phrase ending in a specific nominalized verb form rather than a full sentence — `~함` for completed work, `~할 예정임` for planned work, `~이 필요함` for a request. Which ending is correct depends on what kind of section the item sits in, so "just rewrite it in bullet points" does not get you there.
>
> **And if you're not in Korea, this may still be familiar.** Anyone who has filed a government procurement bid, a regulatory submission, a grant report, or a standards-body filing has met the same problem: the receiving institution has already decided what the document must look like, and "close enough" is not a passing grade. The particulars here are Korean. The shape of the problem is not.

### Existing tools solve the opposite direction

There is no shortage of tooling for Korean documents. But most of it solves **"Markdown → hwpx"** — turning *content into a format*, where the format is the one the tool decided on.

What working professionals need more often runs the other way:

> **"Match this institution's format / last quarter's report — exactly."**

The target format is already fixed, and it was fixed by **the commissioning body or the department above you**, not by your tool. It isn't yours to choose.

### But shipping templates wasn't an option

The easy fix is to bundle a pile of per-institution templates. It fails twice:

1. **Coverage never arrives.** Formats differ across institutions, and differ again by document type *within* an institution. A built-in template list is permanently behind.
2. **They aren't mine to ship.** Putting another organization's form files into a public repository and distributing them is something you simply shouldn't do.

So the design goes the other way: **the format is a runtime input, not code** (bring-your-own-template). You hand over your own sample; the skill extracts the shape from it as data. **This repository contains no institutional form assets of any kind.** Even the bundled [examples/example-walkthrough.md](examples/example-walkthrough.md) is a fictional example, not a real form.

### And generating wasn't enough

Output that's been shaped to a format generally *looks* right. The problem is that **looking right and being right are different things.** Three items left in the wrong sentence register, a sub-item indented two spaces instead of three, a mandatory "Attachments" section missing entirely — none of that jumps out at the eye.

So the skill doesn't stop at generation. It **compares its output against the sample, item by item** (Phase 4): section skeleton, bullet system, sentence endings, date notation, table columns, signature format, each judged and reported. **That comparison, more than the generation, is what this skill is for.**

---

## What makes it different

- 📥 **The format is a runtime input** — works with any institution's format; no institutional assets live in this repo.
- 🔍 **Learn-from-sample** — extracts skeleton, bullet system, register, table conventions and signature format from your sample (using kordoc's format intelligence for hwpx).
- ✅ **Built-in fidelity check** — after generating, it reports item by item whether the format was actually followed.
- 🚫 **Only what was observed** — formatting the parser couldn't read is not invented; it's recorded as `UNOBSERVED`. To assert "body text is 15pt," it has to have actually read 15pt.
- 🔒 **Facts unchanged, personal data not copied** — only the format is transplanted; figures and proper nouns are left alone, and real names in the sample's signature block are not carried into the new document.

---

## How it works

```
[Sample format] ──parse──▶ [Format profile] ──┐
                                              ├──▶ [New document in that format] ──▶ [Fidelity check]
[New content] ────organize────────────────────┘
```

There are **two inputs**, and both are required.

1. **A format sample** — the shape to follow. An institution's distributed form, last quarter's report or plan, an empty form template — any of these. `.hwp` / `.hwpx` / `.docx` (`.pdf` for structural reference only).
2. **The content to place in it** — new copy, notes, or bullet points. A draft already written in some other format works; so does a bare list of items.

If you ask for "our institution's format" without providing a sample, the skill **asks for one rather than inventing a format.**

### Phase 1 — Profile extraction ★ your checkpoint

The sample is parsed into data: document type, section skeleton (hierarchy, order, mandatory sections), the bullet marker and indentation for each level, sentence-ending register, title/date/signature/document-number conventions, table practice, fonts and margins. The parsing path depends on the format:

| Format | Parsing path |
|---|---|
| `.hwp` / `.hwpx` | **kordoc MCP** — `parse_document`, `parse_form`, `extract_profile`, `parse_table`, `detect_format` |
| `.docx` | **python-docx** — bullet level from the leading marker character plus leading-space count; font via a three-step fallback; table shading by reading `w:shd` XML directly; margins in EMU |
| `.pdf` | Text and layout for reference (precise format reproduction is limited) |

**The skill stops here and asks you to confirm:** "Here's the shape I read out of your sample — and here's what I couldn't observe. Shall I match this?" This is the checkpoint that prevents building an entire document against a misread profile and then having to unwind it.

> In non-interactive or single-shot runs (batch jobs, subagents) where there's nobody to answer, it does not block. It writes the profile out as an artifact, proceeds, and collects everything needing confirmation into the Phase 4 checklist as ⚠️.

### Phase 2 — Content mapping

New content is placed into the skeleton and the register rules are applied. Three disciplines matter here:

- **Missing information is not invented.** If the new content has no date or signatory, a placeholder in the correct *format* goes in, flagged ⚠️ for you to confirm.
- **Data is not forced into tables.** If your data's natural column count differs from the sample's table, the skill does not pad blanks with "-" or repeat a value to fill the shape — it asks whether to conform to the sample's columns or build a new table that fits the data. Forcing the fit distorts the data.
- **Figures and proper nouns are untouched.** If converting a sentence ending looks like it would shift the meaning, the original text stays and gets flagged.

When converting noun-phrase notes into *gaejosik*, tense is decided by **which section the item belongs to**. The same note, "call for proposals closed," becomes "closed the call" in a results section and "will close the call" in a plan section (see [references/korean-form-conventions.md](references/korean-form-conventions.md)).

### Phase 3 — Generation

Output is produced in the profile's format. Bullet markers and indentation are attached **at this stage** — they're deliberately kept out of the Phase 2 text so markers can't end up **double-attached**.

- **hwpx sample → hwpx output** — kordoc `fill_form` (for form-field templates) or a style-donor build, preserving the original's formatting.
- **docx sample → docx output** — generated with python-docx, explicitly re-applying the formatting that was **observed** in the original (heading font/size/alignment, cell shading, margins in EMU).

The original sample is never overwritten; output goes to a new file. A Markdown draft can be emitted alongside for review.

### Phase 4 — Fidelity check ★ how to read the result

The output is compared against the sample profile and reported as a table. For hwpx, kordoc `compare_documents` can compare structure directly.

| Item | Sample | Output | Verdict |
|---|---|---|---|
| Section skeleton and order | Ⅰ□ㅇ- hierarchy | identical | ✅ |
| Bullet system | ㅇ 1 space / - 3 / * 5 | identical | ✅ |
| Double-attached markers | — | none | ✅ |
| Gaejosik register | ~함 / ~임 | 3 items still in declarative form | ⚠️ locations given |
| Date notation | '26. 5. 7.(목) | format matches, value is a placeholder | ⚠️ confirm |
| Table columns | 4 columns | 2 columns, built to fit the data | ⚠️ reason given |
| Mandatory sections | includes Attachments | missing | 🔴 |

**How to read the verdicts:**

- ✅ — matches the sample. Nothing to do.
- ⚠️ — a partial match, or **a decision the skill deliberately left to you**. Almost always one of three things: ① an item where the register conversion was ambiguous, so the original wording was kept; ② a placeholder where information (date, signatory) simply wasn't available; ③ a genuine decision point such as a table-column conflict. **This is the column a human needs to read** — answer, and it gets applied.
- 🔴 — missing or in violation. If even one line is 🔴, the skill does not report the document as format-compliant.

If you see `UNOBSERVED`, that is not a failure — it's an **honest report**. It means the parser couldn't read that piece of formatting, and it most often shows up for body font and line spacing in `.docx`. For those fields it's faster to open the original and look.

A full worked run: [examples/example-walkthrough.md](examples/example-walkthrough.md)

---

## Install

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92/form-tailor.git
```

Restart Claude Code and it will pick up relevant requests automatically.

> **A note on language.** The skill's trigger description is written in Korean, so Korean phrasing invokes it most reliably. You can always name the skill directly instead. Its output follows your sample — if the sample is Korean, so is the document.

### Recommended dependency

- **[kordoc](https://github.com/chrisryugj/kordoc) MCP** — the core of HWP/HWPX parsing, form-field filling, and structural comparison. Without it the skill falls back to python-docx / LibreOffice, but **hwpx format preservation is limited**. If your input is `.hwp`/`.hwpx` and kordoc isn't installed, the skill will point you to kordoc or ask for a `.docx` copy instead (noting that LibreOffice conversion can flatten tables).
- `.docx` is handled by python-docx or the docx skill.

---

## How to use it

Provide the sample file and the new content together.

### Scenario 1 — This week's content in last time's shape

```
Format this week's items exactly like the attached weekly report.
The items are in my notes.
```

The most common use. It's the same job as opening last time's file and eyeballing the format — done for you.

### Scenario 2 — Filling an empty form the commissioning body sent

```
Lay this draft out according to the attached form template.
```

The empty form serves as the sample. If it has hwpx form fields, kordoc `fill_form` populates them. If the profile has a **mandatory section** with no corresponding content in your draft, the skill asks what belongs there rather than leaving it blank.

### Scenario 3 — Porting a draft written in some other format

```
Rebuild this Markdown draft in the attached institutional format.
Keep the content as-is — only the format needs to match.
```

"Keep the content as-is" is the default behavior: figures and claims aren't touched, only the format is transplanted. If the content itself needs to be researched or written, that's a different job — see [what this skill doesn't do](#what-this-skill-doesnt-do).

### ★ Intervening at the Phase 1 profile check

When the skill reports "here's the shape I read," that is **the cheapest point at which to correct course.** Just say so:

```
Sub-items are indented two spaces, not three
This one should be in polite declarative form, not gaejosik
The Attachments section is mandatory — don't drop it
Build the table to fit my data instead of the sample's columns
```

Proceeding on a misread profile means rebuilding the whole document. Fixing one line here is far cheaper.

### When the fidelity check comes back with ⚠️

```
The signatory is the team lead — fill it in
Convert the 3 items still in declarative form
Rebuild the table to the sample's 4 columns
```

⚠️ marks decisions the skill held back on, so answering applies them directly.

---

## What this skill doesn't do

Clear boundaries make this easier for everyone.

- **It doesn't produce content.** Research, writing, and fact generation are not this skill's job. If content is missing, it asks rather than filling in.
- **It doesn't change figures or claims.** Only the format is transplanted. If a register conversion looks like it would shift meaning, the original stays and gets flagged.
- **It doesn't invent formats.** It won't start without a sample, and formatting the parser couldn't read is recorded as `UNOBSERVED`.
- **It doesn't copy personal data out of your sample.** Real names and contact details in the signature block are not carried into the new document; the checklist notes it.
- **It doesn't guarantee compliance with any institution's regulations.** It reproduces the formatting **observed** in the sample you provided. For authoritative rules, check that institution's own document-management guidance.

## Limitations

- **Full `.hwp`/`.hwpx` handling depends on kordoc.** Without it you have to route through `.docx`, and formatting can be lost in transit.
- **`.docx` parsing has a hard ceiling.** python-docx doesn't resolve style inheritance, so body font and line spacing frequently come back `None`. Margins and page size are read reliably; fonts and line spacing often end up `UNOBSERVED`. The skill labels this rather than hiding it.
- **`.pdf` samples are structural reference only.** Don't expect precise format reproduction.
- **The fidelity check only compares what made it into the profile.** Formatting that was never observed can't be checked either — a table full of ✅ does not mean every aspect of the format is perfect.
- **The verification record published in this repo is a single smoke test.** v0.2.0 is the release that fixed the defects it exposed — an under-specified `.docx` path, table schema conflicts, missing header information, and the non-interactive fallback — after an adversarial run completed all four phases against a synthetic form. Details in [CHANGELOG.md](CHANGELOG.md).

---

## Repository layout

| File | Contents |
|---|---|
| [SKILL.md](SKILL.md) | The skill itself — the four-phase procedure and its safety/scope principles |
| [references/profile-schema.md](references/profile-schema.md) | Format profile schema (YAML) + per-parser observability table |
| [references/korean-form-conventions.md](references/korean-form-conventions.md) | General Korean official-document conventions + noun-phrase → *gaejosik* conversion table |
| [references/fidelity-checklist.md](references/fidelity-checklist.md) | Phase 4 checklist (separate items for hwpx and docx) |
| [examples/example-walkthrough.md](examples/example-walkthrough.md) | A full four-phase run (fictional form) |

> `korean-form-conventions.md` is a **fallback**. Rules observed in your sample always take precedence over general convention. If your sample is in declarative form, the skill follows the sample.

## Series

Sister tools built on the same design philosophy:

**Agent-team kits** — [policy-research-kit](https://github.com/parkjui92/policy-research-kit) (policy research reports) · [rnd-proposal-kit](https://github.com/parkjui92/rnd-proposal-kit) (Korean government R&D proposals) · [socsci-paper-kit](https://github.com/parkjui92/socsci-paper-kit) (social science papers)

**Authoring kit** — [lecture-deck-kit](https://github.com/parkjui92/lecture-deck-kit) (HTML lecture decks with in-browser live editing)

**Standalone skills** — **form-tailor** (this repo — institutional document formats) · [fact-verify](https://github.com/parkjui92/fact-verify) (source verification) · [paper-proofread](https://github.com/parkjui92/paper-proofread) (Korean academic proofreading) · [report-to-brief](https://github.com/parkjui92/report-to-brief) (report compression)

## License

[MIT](LICENSE). No proprietary institutional forms or template files are included (bring-your-own-template principle).
