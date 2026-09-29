# marketing-skills

A Claude Code plugin for writing marketing copy without tying the method to one brand.

| Skill | What it does |
| --- | --- |
| `/marketing:blog-draft` | SEO brief → full draft → voice check → publish (only after sign-off) |
| `/marketing:social-posts` | One asset → a variant for each channel the repo uses. Drafts only |
| `/marketing:voice-check` | Read-only review against the repo's voice sources, catches voices from other brands leaking in |
| `/marketing:profile` | Create or validate the repo's `.claude/marketing.md` |

## How it splits

The plugin holds only method. Each repo tracks its own **`.claude/marketing.md`**, which
names whose voice to use, which channels to write for, where drafts go and what "publish"
means. It points at existing sources (`path#Heading`) instead of copying them. The
format is in [`plugin/skills/profile/SKILL.md`](plugin/skills/profile/SKILL.md).

Publish adapters:

- `markdown`: the post is a file in git and goes live by PR (e.g. recordsv.lt)
- `datocms`: the draft is markdown, and the repo's own `datocms` skill creates the record (e.g. tomstagl.com)
- `custom`: steps written out in the profile

## Install

Locally, for every repo:

```
/plugin marketplace add tomstagl/marketing-skills
/plugin install marketing@marketing-skills
```

For a repo (including cloud sessions), commit this to its `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "marketing-skills": { "source": { "source": "github", "repo": "tomstagl/marketing-skills" } }
  },
  "enabledPlugins": { "marketing@marketing-skills": true }
}
```

## Guarantees

- Nothing is published, posted or sent without explicit sign-off on that specific piece.
- The `routing` gate in the profile stops a topic from being written in the wrong brand's voice.
- A voice source the session can't read is reported, never skipped silently.
