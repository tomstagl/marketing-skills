---
name: profile
description: Create, validate or explain a repo's marketing profile (.claude/marketing.md) — the file that tells the marketing skills whose voice to write in, which channels exist, where drafts go and how a post gets published (markdown in git, DatoCMS, or custom steps). Use when a marketing skill reports the profile is missing or invalid, or when the user asks to set up marketing, configure the brand voice, add a channel, or change where posts are published. Triggers on: "set up marketing profile", "marketing.md", "configure brand voice", "add a social channel", "change publish target".
argument-hint: [create | check | explain]
---

# Marketing profile

The `marketing` skills (`blog-draft`, `social-posts`, `voice-check`) hold only method.
Everything that belongs to one brand lives in the repo's own `.claude/marketing.md`,
tracked in that repo. This skill creates and validates that file.

**The profile points, it does not copy.** If the repo already has a voice guide, an
SEO section or channel rules (an agent file, a strategy note), the profile references
it as `path#Heading` so there is still one source of truth. Only write rules inline
when no such file exists.

## Format

YAML frontmatter, then optional free prose the skills read as extra context.

```yaml
---
brand: Recordsv.lt                  # required — who is speaking
site: https://recordsv.lt           # required
routing: >-                         # required — which topics belong in this repo
  Recordsv.lt is the subject of the post → here.
  Recordsv.lt is only an example → tomstagl.com.
voice:                              # required — rules, read in full every run
  - .claude/agents/growth-strategist.md#Tone of Voice
samples:                            # optional — published pieces to match the register
  - content/blog/*.md (newest 3)
never_use:                          # optional — other voices that must not leak in
  - ../recordsv-workspace/voice/ (Tom's personal voice — tomstagl.com only)
audience: .claude/agents/growth-strategist.md#Audience
seo:                                # optional — omit to skip keyword research
  - .claude/agents/growth-strategist.md#SEO
  - .claude/agents/growth-strategist.md#SEO Operations
links: >-                           # optional — how tracked links are built
  buildUtm() from src/lib/utm.ts; taxonomy in growth-strategist.md#UTM Taxonomy
blog:
  adapter: markdown                 # markdown | datocms | custom
  draft_path: content/blog/{slug}.md
  brief_path: content/blog/{slug}-brief.md
  format_example: content/blog/<an existing post>.md   # frontmatter to copy
  length: 800-1200 words
  publish: >-                       # what "publish" means here, in plain words
    Commit the file on a branch and open a PR; merging publishes it.
social:
  path: growth/social/{date}-{slug}.md
  channels:
    - name: Reddit
      rules: .claude/agents/growth-strategist.md#Social Formatting
    - name: Instagram
      rules: .claude/agents/growth-strategist.md#Social Formatting
experiments: growth/memory/experiments.md   # optional — omit to skip logging
approval: required                  # publishing always waits for explicit sign-off
---
```

### Adapters

- **`markdown`** — the post is a file in git. `draft_path` is where it lands and
  `publish` says how it goes live (usually: a PR). The skill never merges.
- **`datocms`** — the post is a DatoCMS record. Drafts go to `draft_path` as markdown
  first. Publishing hands the approved draft to the repo's own `datocms` skill or
  command (it holds the model IDs, fields and upload quirks). Set `blog.cms_skill` if
  it has another name.
- **`custom`** — `publish` holds the exact steps, followed literally.

### Paths outside the repo

A path into a sibling repo (`../recordsv-workspace/voice/`) works on a local machine
only. In a cloud session that directory is not there. Give such entries a fallback the
skills can reach anywhere, such as a published URL (`https://tomstagl.com/blog`) or a
GitHub path (`tomstagl/recordsv-workspace:voice/`, read through the GitHub MCP). The
skills try the local path first, then the fallback. If neither works they say which
sources they could not read. They never carry on silently.

## Modes (`$ARGUMENTS`)

### `create` (default when no profile exists)

1. Survey the repo for existing sources: `CLAUDE.md`, `.claude/agents/*.md` (look for
   Tone of Voice / Voice Guidelines / Audience / SEO / Channels headings), strategy
   notes, `content/`, a CMS skill or command, a `utm` helper, an experiments log.
2. Draft the profile, pointing at what you found. Mark anything you guessed with
   `# TODO confirm`.
3. Show it to the user and ask about every TODO, especially `routing` and `never_use`.
   Those two keep one brand's voice out of another's copy. Write the file only after they answer.

### `check`

Parse the frontmatter. For every path and `path#Heading`, confirm the file exists and
the heading is present. Report a table of `field | reference | ok / missing / fallback-only`.
Fail the check if a required field is empty or a `voice` source cannot be read at all.

### `explain`

Summarise in five lines what the marketing skills will do in this repo: whose voice,
which channels, where drafts land and what publishing means.
