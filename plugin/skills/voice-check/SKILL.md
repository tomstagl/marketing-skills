---
name: voice-check
description: Review marketing or blog copy against the current repo's brand voice (sources listed in .claude/marketing.md) and report violations with on-voice rewrites. Also catches the wrong voice leaking in from another brand. Read-only. Use when the user asks to voice-check, brand-check or tone-check copy, review a draft, or confirm something sounds like us. Also run by blog-draft and social-posts before they save. Triggers on: "voice check", "is this on brand", "tone check", "does this sound like me", "does this sound like us", "review this copy".
argument-hint: [file path | --staged | inline text]
---

# Voice check

Review copy against the voice this repo's profile names. **You do not invent voice
rules.** Every rule you apply comes from a source listed in `.claude/marketing.md`, and
you cite it by file and heading.

## 0. Load the profile

Read `.claude/marketing.md`. If it is missing, stop and run the `profile` skill
(`create`). Then read **in full**, every run:

- every `voice` source (a `path#Heading` means that section, to the next heading of
  the same level)
- `samples`, if any: they show the register the rules describe
- `seo` and `links`, only when the copy has a title, meta description or links

If a source cannot be read locally, try its fallback. List any source you still could
not read at the top of the report. A check run against half the rules must say so.

## 1. Resolve the input (`$ARGUMENTS`)

1. A path to an existing file → that file's copy.
2. `--staged` → added or changed prose in `git diff --staged` (copy files only, not code).
3. Empty → copy changed in `git diff`; if none, ask for a path or pasted text.
4. Anything else → treat it as inline copy.

## 2. Check

Work line by line. For each issue record the exact text, the rule it breaks (with its
source) and why it lands wrong with the audience in the profile.

Always check, even if the sources don't spell it out:

- **Wrong voice.** Compare against `never_use` and `routing`. Copy that sounds like
  another brand's voice, or a topic that `routing` sends to another repo, is a
  **blocking** issue. Two brands blending into one voice is the failure this check
  exists to prevent.
- **Generic filler**: superlatives, hype, "in today's fast-paced world", claims with
  no concrete detail behind them.
- **Unsupported claims**: numbers, results or capabilities the copy states and nothing
  in the repo or source material backs.

Check mechanics only where they apply: meta description length and title pattern
from `seo`; tracked links built the way `links` says; email subject rules if the voice
source has them.

## 3. Rewrite

For each flagged line, write an on-voice rewrite that keeps the author's intent. Keep
it concrete and specific. Don't fix what isn't broken.

## Output

```markdown
## Voice check — <file or "inline copy"> · voice: <brand>

**Verdict:** ✅ On voice · ⚠️ Minor fixes · ❌ Off voice (N blocking)
**Sources read:** <list> · **Unreadable:** <list or "none">

| Snippet | Rule (source) | Why it's off | Rewrite |
| ------- | ------------- | ------------ | ------- |

**Mechanics:** <meta length, title pattern, links — or "n/a">

**Summary:** <ready to ship? the single most important fix>
```

If the copy is clean, say so and stop. Don't manufacture nitpicks. This skill is
read-only. Apply fixes only when the user, or the calling skill, asks for them.
