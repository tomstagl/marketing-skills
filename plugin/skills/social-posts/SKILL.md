---
name: social-posts
description: Turn a blog post, release, feature or topic into platform-tailored social posts (LinkedIn, X/Twitter thread, Reddit, forums, Instagram, whatever channels the repo's marketing profile lists), in that repo's brand voice. Drafts only, never posts. Use when the user asks to write a LinkedIn post, a tweet or thread, a Reddit or community post, an Instagram caption, or to share or promote a post or launch socially. Triggers on: "LinkedIn post", "tweet", "X thread", "Reddit post", "social post", "social variants", "promote this post", "share this launch".
argument-hint: [draft path | URL | topic]
---

# Social posts

One source asset → one variant per channel in the profile. Every variant is on voice,
follows that channel's rules, and carries one concrete detail. This skill drafts. A
human posts.

## 0. Load the profile

Read `.claude/marketing.md`. Missing or invalid → stop and run the `profile` skill.
Apply the `routing` gate the same way `blog-draft` does. Read in full: the `voice`
sources, 1–2 `samples`, `social.channels[*].rules`, `links`, and the last 10 social
entries of `experiments` (if set). Note which angles worked and which were archived.

## 1. Read the asset

- **A path** → read the file. **A URL** → fetch it. **A topic** → ask whether a
  published asset exists first. Social posts without anything behind them are
  usually thin.
- Extract the facts the variants may use: the headline insight, the most specific
  concrete details (numbers, names, dates, a surprising finding) and the canonical
  URL. **Use only these facts.** A variant never states a number the asset doesn't.

## 2. Write the variants

One section per entry in `social.channels`, in profile order. For each:

- Apply that channel's `rules` source literally: format, link placement, hashtags,
  flair, length.
- Lead with the insight or the detail, not the product or the author.
- Build the link the way `links` says (UTM helper, taxonomy). If `links` isn't set,
  use the plain canonical URL.
- A channel with no rules source gets its platform's norms: LinkedIn = hook in
  the first two lines, short paragraphs, no link in the body if the profile says so,
  one closing question at most. X = one tweet, plus a thread only if the asset
  carries 4+ distinct points.

Add a 1-line rationale per variant: why this angle suits this channel, and what
engagement would show it worked.

## 3. Voice check

Run the `voice-check` skill on the whole file and fix blocking issues before saving.

## 4. Save and log

Write to `social.path` with a header in this shape:

```markdown
# Social variants — <asset title>

Post: <canonical URL> (<CMS record ID or file path>, <status/date>)
Facts used: <the extracted details, so a reviewer can check every number>
```

If `experiments` is set, append one entry (`channel: social`), listing each variant's
angle and the UTM-tracked metric, with `result: pending`.

Finish by telling the user the file path and which variant to post first, and why.
Never post, schedule or send anything yourself.
