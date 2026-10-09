# form-tailor

[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skill-purple.svg)

[한국어](README.md) · **English**

A [Claude Code](https://claude.com/claude-code) skill that makes your documents match an organization's required format.

**Show it one sample document and it learns the shape of it**, then builds your new content in that same shape. It ships with no built-in templates, so any organization's format works. And once it's finished, it **goes back through what it made and tells you, item by item, whether it really matches.**

## What makes it different

Some background first. Official paperwork in Korea runs on HWP and HWPX files — the formats of a word processor called Hangul, which is the de facto standard for institutional paperwork there, the way `.docx` belongs to Word. Government offices, public agencies and universities each keep their own house format for those files: which sections come in what order, which small symbol marks each level of bullet, how sentences are supposed to end. The format you have to match is theirs, not yours.

If that sounds like a local quirk, it isn't. Anyone who has filled in a government procurement form, a grant application or a funder's standard reporting template has met the same problem in a different file format: someone else decided the layout, and matching it exactly is your job.

There is already plenty of tooling for Korean documents. But most of it solves "turn my Markdown into a Hangul file" — pouring content into **the format the tool picked**. What people need more often runs the other way: the format is already fixed, and it was fixed by **the commissioning body or the department above you.**

So why not just bundle every organization's template? Two reasons that can't work. Formats differ between organizations, and differ again by document type *inside* one organization, so a bundled list is always behind. And another organization's form files simply **aren't mine to publish.** So this was built the other way around — **you supply the format at the moment you use it.** Which is why this repository contains no organization's form files of any kind.

→ [Why I built this, and fuller usage notes](docs/why.md)

## How it works

```
Your sample + your content → learn the shape → ★you confirm it → build → 🚦check it matches
```

You give it **two things**, and both are required: a **sample to copy** (`.hwp`, `.hwpx` or `.docx`; a `.pdf` works for structure only) and the **content to put in it** (new copy, notes, or a draft you already wrote in some other format). Ask for "our organization's format" without attaching a sample and it **asks you for one rather than inventing a format.**

**Step 1 — it reads the shape out of your sample. ★** What order the sections come in, which symbol marks each level of bullet (□, then ㅇ, then -, and so on), how sentences end, how the title, date and signature block are laid out, how tables are used. **Then it stops and shows you what it found.**

**Step 2 — it fits your content into that shape.** Your material goes into the section order and takes on the sample's sentence style. It doesn't invent information it wasn't given, doesn't force your data into the sample's tables, and doesn't touch your figures or proper nouns.

**Step 3 — it builds the document**, re-applying the formatting it actually read out of the sample. Your original sample file is never overwritten.

**Step 4 — it compares the result against the sample** and reports back as a table.

## Then it checks its own work

A document that's been shaped to a format generally *looks* right. The trouble is that **looking right and being right are two different things.** Three items left in the wrong sentence style, a sub-item indented one space off, a required "Attachments" section missing entirely — none of that catches the eye. **That comparison, more than the building, is what this skill is for.**

- **✅ matches the sample.**
- **⚠️ a partial match, or a decision it deliberately left to you** — an item whose style conversion was ambiguous, a blank left where information wasn't available, a table whose columns don't line up. **This is the part a person needs to read**, and telling it what you want applies your answer directly.
- **🔴 missing, or in violation.** If even one line is 🔴, it does not report the document as matching the format.

## Install

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/parkjui92/form-tailor.git
```

Restart Claude Code and it will pick up relevant requests on its own. Its trigger description is written in Korean, so Korean phrasing invokes it most reliably — otherwise just name the skill directly. The output follows your sample, whatever language that's in.

## Using it

Just ask in plain language.

```
Format this week's items exactly like the attached weekly report   ← reuse last time's shape
Lay this draft out according to the attached blank form            ← fill in a form you were sent
Rebuild this Markdown draft in the attached organization's format  ← move a draft across
Sub-items are indented two spaces, not three                       ← ★when it shows you the shape
Fix the 3 items still in the wrong sentence style                  ← when the check comes back ⚠️
```

It stops once, right after it reads the shape. Building a whole document on a misread shape and undoing it later costs far more than **fixing one line at that point.**

## Good to know

- Reading and writing `.hwp` / `.hwpx` files needs a separate tool for handling Hangul files, called [kordoc](https://github.com/chrisryugj/kordoc). Without it the skill falls back to the Word-file tooling, but **keeping Hangul formatting intact is limited that way.**
- **It doesn't write your content, and it doesn't invent formats.** Research and writing aren't its job, and it won't change your figures or claims — it moves the format across, nothing else. Real names and contact details in the sample's signature block aren't copied over either; it flags them instead.
- **It can't promise you've complied with any organization's rules.** It reproduces the formatting it **actually observed** in the sample you gave it.
- **A table full of ✅ doesn't mean the formatting is perfect.** With `.docx` samples, body font and line spacing often can't be read at all, and **formatting that was never read can't be checked either.** `.pdf` samples are for structure only.
- The published verification record is a single run that took all four steps to the end against a made-up form ([CHANGELOG.md](CHANGELOG.md)).
- More detail: [SKILL.md](SKILL.md) · [what goes into the shape it reads](references/profile-schema.md) · [Korean official-document style conventions](references/korean-form-conventions.md) · [what gets checked](references/fidelity-checklist.md)

## Related work


My other tools, mapped by research stage, are on [my profile](https://github.com/parkjui92).

## License

[MIT](LICENSE). No organization's forms or template files are included.
