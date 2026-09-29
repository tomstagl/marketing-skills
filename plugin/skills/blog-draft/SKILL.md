---
name: blog-draft
description: Write an SEO content brief and/or a full blog post draft in the current repo's brand voice, save it where the repo's marketing profile says, and — only after explicit sign-off — publish it through the repo's adapter (markdown file + PR, or DatoCMS). Use when the user asks to write a blog post or article, draft a post about a topic, produce an SEO brief or outline, target a keyword, or publish an approved draft. Triggers on: "write a blog post", "draft a post", "blog draft", "SEO brief", "article about X", "content for keyword", "publish this post".
argument-hint: [topic or keyword] [--brief] | publish [draft path]
---

# Blog draft

Brief → draft → voice check → (sign-off) → publish. All brand specifics come from
`.claude/marketing.md`; this file is only the method.

## 0. Load the profile

Read `.claude/marketing.md`. Missing or invalid → stop and run the `profile` skill.

**Routing gate.** Test the topic against `routing` before you write anything. If the
topic belongs in another repo, say which one and stop. Never draft it here in this
brand's voice.

Read in full: the `voice` sources, 2–3 `samples`, `audience`, `seo` (if set),
`blog.format_example`, and the last 10 relevant entries of `experiments` (if set).
List the existing posts under `blog.draft_path`'s directory, or the CMS's list for
`datocms`, so you don't duplicate a covered topic.

## 1. Brief

If `seo` is set, research keywords (WebSearch plus the SEO sources): the primary
keyword, 2–3 secondary ones, search intent and funnel stage. If `seo` is not set,
skip the keyword fields; the brief still sets the angle.

```markdown
## Brief: <working title>
**Angle:** <the one thing this post says that others don't — grounded in real experience or data>
**Audience:** <from profile>
**Target keyword / secondary:** <or "n/a">
**Intent / funnel stage:** <or "n/a">
**Length:** <blog.length>
**Title:** <option A> · **Alt:** <option B>
**Outline:** 1. … 2. … 3. …
**Concrete details to include:** <names, numbers, dates, examples — each with its source>
**Internal links:** <2–3 existing pages/posts>
**CTA:** <one, verb-led>
```

With `--brief`: save it to `blog.brief_path`, skip to step 4, then stop.

## 2. Draft

- Follow the outline. Copy the frontmatter shape of `blog.format_example` exactly
  (field names, date format, image fields). Set `status: draft` if the format has it.
- Open with the specific detail or insight, not the setup.
- Every factual claim traces to a source: the repo, a URL you fetched, or something
  the user said. If there isn't enough material to write with authority, say so and
  propose a narrower angle rather than padding.
- No placeholders or TODOs in the body. The draft should be publishable as written.

Save to `blog.draft_path`. Images you can't produce go in the frontmatter as a
description (`<file> (<size>) — alt: …`), not as a broken link.

## 3. Voice check

Run the `voice-check` skill on the saved draft. Fix every blocking issue and re-run
until the verdict is ✅ or ⚠️. Tell the user about any ⚠️ items you chose to keep.

## 4. Log (only if `experiments` is set)

Append an entry in the shape the existing log already uses, with `channel: seo` (or
`blog`), the hypothesis, the title A/B variants, the metric, and `result: pending`.

## 5. Publish — only on explicit sign-off

Stop after the draft and show the user: title, path, word count, the voice-check
verdict and the brief's angle. **Do not publish until the user approves this specific
draft.** Approval of an earlier draft or another post doesn't count. With `approval:
required` there are no exceptions, including for scheduled or agent runs, which leave
the draft for a human.

After sign-off, by `blog.adapter`:

- **`markdown`**: do what `blog.publish` says (usually: a branch, a commit of the
  file, a PR). Never merge. Report the PR URL.
- **`datocms`**: invoke the repo's CMS skill (`blog.cms_skill`, default `datocms`)
  with the approved draft. Create the record, upload images, then **confirm
  once more before the publish call.** Report the record ID and live URL, and write
  them back into the draft file's header (as `.claude/content/*-social.md` headers do).
- **`custom`**: follow `blog.publish` literally.

When invoked as `publish <path>`, skip steps 1–4. Voice-check the file, then go
straight to this step.
