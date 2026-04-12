# Engineering Blog — Design Spec

**Date:** 2026-04-12
**Status:** Approved

---

## Overview

Add a second blog section to the existing groupscout-blog Astro site, focused on engineering. The build-log blog at `/blog/` documents the product journey. The engineering blog at `/engineering/` documents technical decisions, system architecture, and implementation details — drawn from the `backend-docs/` directory and written to the standards of Strunk's *Elements of Style*.

---

## Architecture

### New files

```
src/content/engineering/              ← markdown posts (AI-drafted from backend-docs)
src/pages/engineering/
  index.astro                         ← engineering index, dense post list
  [...slug].astro                     ← individual post route
src/layouts/EngineeringPost.astro     ← post layout with indigo accent, tighter spacing
```

### Content collection

A new `engineering` collection defined in `src/content/config.ts` alongside the existing `blog` collection. Schema:

```ts
engineering: defineCollection({
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    tags: z.array(z.string()).optional(),
    source: z.string().optional(), // e.g. "ARCHITECTURE.md"
    draft: z.boolean().optional(),
  }),
}),
```

The `source` field records which backend-doc the post was generated from. It appears as a muted attribution line below the date in the post header.

### Routing

| Path | File | Purpose |
|---|---|---|
| `/engineering/` | `src/pages/engineering/index.astro` | Post list |
| `/engineering/[slug]` | `src/pages/engineering/[...slug].astro` | Individual post |

---

## Visual Design

The engineering section shares the zinc base palette and dark/light mode system of the current blog, but is visually distinct:

| Property | Build-log blog | Engineering blog |
|---|---|---|
| Accent color | zinc | indigo (`indigo-500` light / `indigo-400` dark) |
| Body font size | `text-base` | `text-sm` |
| Line height | relaxed | normal (tighter) |
| Post list layout | Card grid (3-col) | Single-column list with `<hr>` separators |
| Code blocks | Inherits theme | Dark background in both light and dark mode |
| Index hero | Full hero section | One-line description, list immediately below |
| Nav active indicator | zinc underline | indigo underline |

The `EngineeringPost.astro` layout wraps the existing `BaseLayout.astro` (for nav, footer, theme toggle) and applies the indigo accent and tighter typography via scoped styles or Tailwind classes.

---

## Navigation

The global `Navigation.astro` component gains an "Engineering" link pointing to `/engineering/`. It follows the same link style as existing nav items but highlights with an indigo underline when active.

---

## Content Generation Workflow

1. User specifies a source doc (e.g. "draft a post from `ARCHITECTURE.md`")
2. Claude reads the source doc and `prompts/eos/elements-of-style.md`
3. Claude drafts one or more posts in `src/content/engineering/`, applying EOS principles: active voice, definite language, omit needless words
4. Frontmatter includes `source:` field referencing the originating doc
5. User reviews the draft(s) and directs: approve as-is, split into multiple posts, merge with another doc, or revise
6. Approved posts are committed and go live

A single backend-doc may produce one post or several — the user decides after reviewing the draft.

---

## Out of Scope

- A separate Astro project or deploy target
- Custom syntax highlighting theme (use existing Astro defaults)
- Search or filtering on the engineering index (can be added later)
- Comments or reactions
