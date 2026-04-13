# Engineering Blog Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `/engineering/` section to the existing Astro site — a separate content collection with a visually distinct indigo-accented layout, fed by AI-generated posts from `backend-docs/`.

**Architecture:** New `engineering` Astro content collection alongside `blog`; dedicated `EngineeringPost.astro` layout wrapping the existing `Layout.astro`; new pages at `src/pages/engineering/`. No shared state with the blog collection.

**Tech Stack:** Astro 5, Tailwind CSS 3, TypeScript, `reading-time` (already installed)

---

## File Map

| Action | File | Responsibility |
|---|---|---|
| Modify | `src/content/config.ts` | Add `engineering` collection schema |
| Modify | `src/components/Navigation.astro` | Add Engineering nav item; fix active-link detection |
| Create | `src/layouts/EngineeringPost.astro` | Post layout: indigo accent, tight prose, source attribution |
| Create | `src/pages/engineering/index.astro` | Dense single-column post list |
| Create | `src/pages/engineering/[...slug].astro` | Individual post route with prev/next nav |
| Create | `src/content/engineering/` | Directory for engineering posts (empty to start) |

---

## Task 1: Add engineering collection to content config

**Files:**
- Modify: `src/content/config.ts`

- [ ] **Step 1: Open `src/content/config.ts` and replace its contents**

```typescript
import { defineCollection, z } from 'astro:content';

const blogCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    heroImage: z.string().optional(),
    tags: z.array(z.string()).optional(),
    readingTime: z.string().optional(),
    draft: z.boolean().optional(),
  }),
});

const engineeringCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    tags: z.array(z.string()).optional(),
    source: z.string().optional(),
    readingTime: z.string().optional(),
    draft: z.boolean().optional(),
  }),
});

export const collections = {
  'blog': blogCollection,
  'engineering': engineeringCollection,
};
```

- [ ] **Step 2: Create the engineering content directory**

Create an empty directory `src/content/engineering/` — Astro requires the directory to exist even if there are no posts yet. Add a `.gitkeep` file:

```
src/content/engineering/.gitkeep
```

(Empty file, no content.)

- [ ] **Step 3: Verify TypeScript is happy**

Run:
```bash
npx astro check
```
Expected: no errors related to collections.

- [ ] **Step 4: Commit**

```bash
git add src/content/config.ts src/content/engineering/.gitkeep
git commit -m "feat(content): add engineering collection schema"
```

---

## Task 2: Update navigation

**Files:**
- Modify: `src/components/Navigation.astro`

- [ ] **Step 1: Update `navItems` array and active-link detection**

In `src/components/Navigation.astro`, replace the frontmatter block (lines 1–16) with:

```astro
---
import ThemeToggle from './ThemeToggle.astro';
const BASE_URL = import.meta.env.BASE_URL;

const navItems = [
  { text: 'Home', href: `${BASE_URL}` },
  { text: 'Blog', href: `${BASE_URL}blog/` },
  { text: 'Engineering', href: `${BASE_URL}engineering/` },
  { text: 'Tags', href: `${BASE_URL}tags/` },
  { text: 'About', href: `${BASE_URL}about/` },
  { text: 'RSS', href: `${BASE_URL}rss.xml` },
];

// Get current path for active link highlighting
const pathname = new URL(Astro.request.url).pathname;
const currentPath = pathname.slice(1); // remove leading "/"
---
```

- [ ] **Step 2: Update the `isActive` computation in both the desktop nav and mobile nav**

The current code has `isActive` computed inline in two `.map()` calls. Update both instances.

Find this pattern (appears twice):
```js
const isActive = currentPath === (item.href === '/' ? '' : item.href.slice(1));
```

Replace both occurrences with:
```js
const isActive = item.href === BASE_URL
  ? currentPath === ''
  : currentPath.startsWith(item.href.slice(1));
```

This makes "Engineering" stay highlighted when reading any engineering post, and likewise for Blog and Tags.

- [ ] **Step 3: Start dev server and verify**

Run:
```bash
npm run dev
```

Open `http://localhost:4321`. Confirm "Engineering" appears in the nav. Click it — you'll get a 404 (the page doesn't exist yet), which is expected. Confirm the active highlight works on existing pages (Home, Blog, Tags, About).

- [ ] **Step 4: Commit**

```bash
git add src/components/Navigation.astro
git commit -m "feat(nav): add Engineering link and fix startsWith active detection"
```

---

## Task 3: Create EngineeringPost layout

**Files:**
- Create: `src/layouts/EngineeringPost.astro`

- [ ] **Step 1: Create the file with this content**

The layout uses two slots: the default slot for prose content, and a named `after` slot for anything that should appear outside the prose wrapper (prev/next nav). This prevents prose styles from leaking into the navigation.

```astro
---
import Layout from './Layout.astro';
import FormattedDate from '../components/FormattedDate.astro';

export interface Props {
  title: string;
  description: string;
  pubDate: Date;
  updatedDate?: Date;
  tags?: string[];
  source?: string;
  readingTime?: string;
}

const {
  title,
  description,
  pubDate,
  updatedDate,
  tags = [],
  source,
  readingTime = '5 min read',
} = Astro.props;
---

<Layout title={`${title} | Engineering`} description={description}>
  <article class="max-w-2xl mx-auto px-4 sm:px-6 py-10 sm:py-16">
    <header class="mb-8 pb-6 border-b border-zinc-100 dark:border-zinc-800">
      <h1 class="text-2xl sm:text-3xl font-bold tracking-tight text-zinc-900 dark:text-zinc-100 mb-1 engineering-title">
        {title}
      </h1>

      <div class="mt-3 flex flex-wrap items-center gap-x-3 gap-y-1 text-xs text-zinc-500 dark:text-zinc-400">
        <FormattedDate date={pubDate} />
        {readingTime && <span>· {readingTime}</span>}
        {source && (
          <span class="font-mono bg-zinc-100 dark:bg-zinc-800 px-2 py-0.5 rounded">
            {source}
          </span>
        )}
      </div>

      {tags.length > 0 && (
        <div class="mt-3 flex flex-wrap gap-2">
          {tags.map(tag => (
            <span class="inline-flex items-center rounded-full bg-indigo-50 dark:bg-indigo-950/30 px-2.5 py-0.5 text-xs font-medium text-indigo-700 dark:text-indigo-300">
              #{tag}
            </span>
          ))}
        </div>
      )}
    </header>

    <!-- Default slot: prose content only -->
    <div class="engineering-prose prose prose-sm prose-zinc dark:prose-invert max-w-none">
      <slot />
    </div>

    {updatedDate && (
      <p class="mt-8 text-xs text-zinc-400 dark:text-zinc-500 italic">
        Updated: <FormattedDate date={updatedDate} />
      </p>
    )}

    <!-- Named slot: prev/next nav and anything outside prose -->
    <slot name="after" />
  </article>
</Layout>

<style>
  /* Indigo underline accent on the title */
  .engineering-title {
    border-bottom: 2px solid #6366f1;
    display: inline-block;
    padding-bottom: 0.2rem;
  }

  :global(.dark) .engineering-title {
    border-color: #818cf8;
  }

  /* Dark code blocks in both light and dark mode */
  .engineering-prose :global(pre) {
    background-color: #1e293b !important;
  }

  .engineering-prose :global(pre code) {
    background-color: transparent !important;
    color: #e2e8f0 !important;
  }

  /* Inline code */
  .engineering-prose :global(code:not(pre code)) {
    background-color: #1e293b;
    color: #e2e8f0;
    padding: 0.15rem 0.35rem;
    border-radius: 0.25rem;
    font-size: 0.8em;
  }

  /* Indigo links */
  .engineering-prose :global(a) {
    color: #4f46e5;
    text-underline-offset: 3px;
  }

  :global(.dark) .engineering-prose :global(a) {
    color: #818cf8;
  }

  /* Tighter paragraph spacing */
  .engineering-prose :global(p) {
    margin-bottom: 0.875rem;
    line-height: 1.6;
  }
</style>
```

- [ ] **Step 2: Commit**

```bash
git add src/layouts/EngineeringPost.astro
git commit -m "feat(layout): add EngineeringPost layout with indigo accent"
```

---

## Task 4: Create engineering index page

**Files:**
- Create: `src/pages/engineering/index.astro`

- [ ] **Step 1: Create the file**

```astro
---
import { getCollection } from 'astro:content';
import Layout from '../../layouts/Layout.astro';

const now = new Date();
const allPosts = (await getCollection('engineering'))
  .filter(post => post.data.pubDate <= now && post.data.draft !== true)
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---

<Layout title="Engineering | Group Scout" description="Technical notes on how Group Scout is built.">
  <div class="max-w-2xl mx-auto px-4 sm:px-6 py-10 sm:py-16">
    <div class="mb-10">
      <h1 class="text-2xl font-bold tracking-tight text-zinc-900 dark:text-zinc-100 mb-2">
        Engineering
      </h1>
      <p class="text-sm text-zinc-500 dark:text-zinc-400">
        Technical notes on how Group Scout is built.
      </p>
    </div>

    {allPosts.length === 0 ? (
      <p class="text-sm text-zinc-400 dark:text-zinc-500">No posts yet.</p>
    ) : (
      <div class="divide-y divide-zinc-100 dark:divide-zinc-800">
        {allPosts.map(post => (
          <article class="py-5 group relative">
            <div class="flex flex-col sm:flex-row sm:items-baseline gap-1 sm:gap-5">
              <time
                datetime={post.data.pubDate.toISOString()}
                class="shrink-0 text-xs font-mono text-zinc-400 dark:text-zinc-500"
              >
                {post.data.pubDate.toLocaleDateString('en-US', {
                  year: 'numeric',
                  month: 'short',
                  day: 'numeric',
                })}
              </time>

              <div class="flex-1 min-w-0">
                <h2 class="text-sm font-semibold text-zinc-900 dark:text-zinc-100 group-hover:text-indigo-600 dark:group-hover:text-indigo-400 transition-colors duration-150">
                  <a
                    href={`${import.meta.env.BASE_URL}engineering/${post.slug}/`}
                    class="before:absolute before:inset-0"
                  >
                    {post.data.title}
                  </a>
                </h2>

                <p class="mt-0.5 text-xs text-zinc-500 dark:text-zinc-400 line-clamp-2">
                  {post.data.description}
                </p>

                {post.data.source && (
                  <span class="mt-1 inline-block text-xs font-mono text-zinc-400 dark:text-zinc-500">
                    {post.data.source}
                  </span>
                )}
              </div>
            </div>
          </article>
        ))}
      </div>
    )}
  </div>
</Layout>
```

- [ ] **Step 2: Commit**

```bash
git add src/pages/engineering/index.astro
git commit -m "feat(pages): add /engineering index page"
```

---

## Task 5: Create engineering slug page

**Files:**
- Create: `src/pages/engineering/[...slug].astro`

- [ ] **Step 1: Create the file**

`<Content />` goes into the default slot (inside the prose wrapper). The prev/next nav goes into the named `after` slot (outside the prose wrapper).

```astro
---
import { getCollection } from 'astro:content';
import EngineeringPost from '../../layouts/EngineeringPost.astro';
import readingTime from 'reading-time';

export async function getStaticPaths() {
  const entries = await getCollection('engineering');
  const now = new Date();

  const published = entries.filter(
    e => e.data.pubDate <= now && e.data.draft !== true
  );

  const sorted = [...published].sort(
    (a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf()
  );

  return sorted.map((entry, index) => {
    const rt = readingTime(entry.body);
    const minutes = Math.max(1, Math.ceil(rt.minutes));
    return {
      params: { slug: entry.slug },
      props: {
        entry,
        readingTimeValue: `${minutes} min read`,
        prevPost: index < sorted.length - 1 ? sorted[index + 1] : null,
        nextPost: index > 0 ? sorted[index - 1] : null,
      },
    };
  });
}

const { entry, readingTimeValue, prevPost, nextPost } = Astro.props;
const { Content } = await entry.render();
const finalReadingTime = entry.data.readingTime || readingTimeValue;
---

<EngineeringPost {...entry.data} readingTime={finalReadingTime}>
  <!-- Default slot: rendered markdown, inside the prose wrapper -->
  <Content />

  <!-- Named slot: prev/next nav, rendered outside the prose wrapper -->
  <div slot="after" class="mt-12 pt-8 border-t border-zinc-200 dark:border-zinc-800 grid grid-cols-1 sm:grid-cols-2 gap-4">
    {prevPost && (
      <a
        href={`${import.meta.env.BASE_URL}engineering/${prevPost.slug}/`}
        class="group flex flex-col p-4 rounded-lg border border-zinc-200 dark:border-zinc-800 hover:border-indigo-300 dark:hover:border-indigo-700 hover:bg-indigo-50/50 dark:hover:bg-indigo-950/20 transition-all duration-200"
      >
        <span class="text-xs text-zinc-400 dark:text-zinc-500 mb-1">← Previous</span>
        <span class="text-sm font-medium text-zinc-900 dark:text-zinc-100 group-hover:text-indigo-600 dark:group-hover:text-indigo-400 transition-colors line-clamp-2">
          {prevPost.data.title}
        </span>
      </a>
    )}
    {nextPost && (
      <a
        href={`${import.meta.env.BASE_URL}engineering/${nextPost.slug}/`}
        class="group flex flex-col p-4 rounded-lg border border-zinc-200 dark:border-zinc-800 hover:border-indigo-300 dark:hover:border-indigo-700 hover:bg-indigo-50/50 dark:hover:bg-indigo-950/20 transition-all duration-200 sm:text-right sm:col-start-2"
      >
        <span class="text-xs text-zinc-400 dark:text-zinc-500 mb-1">Next →</span>
        <span class="text-sm font-medium text-zinc-900 dark:text-zinc-100 group-hover:text-indigo-600 dark:group-hover:text-indigo-400 transition-colors line-clamp-2">
          {nextPost.data.title}
        </span>
      </a>
    )}
  </div>
</EngineeringPost>
```

- [ ] **Step 2: Commit**

```bash
git add src/pages/engineering/[...slug].astro
git commit -m "feat(pages): add /engineering/[slug] post route"
```

---

## Task 6: Add a sample post and verify full build

**Files:**
- Create: `src/content/engineering/groupscout-architecture.md`

- [ ] **Step 1: Create a minimal sample post to exercise the collection**

```markdown
---
title: 'System Architecture Overview'
description: 'How Group Scout is structured: collector, enrichment, storage, and notification layers.'
pubDate: '2026-04-12'
tags: ['architecture', 'go', 'system-design']
source: 'ARCHITECTURE.md'
---

Group Scout is a Go backend structured in four layers. Each layer has one job.

## Collector

The collector layer polls public data sources — building permit databases, BC Bid RSS feeds, the Vancouver Convention Centre events page, Google News, and Eventbrite. Each source is a separate struct implementing a common interface. The pipeline runs on a schedule or via HTTP trigger (for n8n integration).

## Storage

All raw and enriched records land in Postgres. The schema is simple: one leads table with a JSONB column for source-specific metadata. Deduplication runs at write time using a hash of the source URL and project identifier.

## Enrichment

The enrichment layer takes a raw lead and produces a score. Two enrichers run in sequence: a rule-based pre-scorer (keyword matching, project type, dollar value) and a Claude API call that estimates crew size, project duration, and probability of out-of-town workers. The Claude call returns a priority score (1–10) and a plain-English rationale.

## Notification

Scored leads above a threshold trigger a Slack message immediately. A weekly digest runs every Monday morning and summarizes the top leads from the prior week. The `alertd` binary handles a separate use case: airport disruption monitoring that generates hotel-targeted outreach copy.
```

- [ ] **Step 2: Remove the `.gitkeep` placeholder**

```bash
rm src/content/engineering/.gitkeep
```

- [ ] **Step 3: Run a full build to verify everything compiles**

```bash
npm run build
```

Expected output ends with something like:
```
 generating static routes
  ├─ /engineering/ (+Xms)
  ├─ /engineering/groupscout-architecture/ (+Xms)
  ...
 build complete in Xs
```

If the build fails, read the error — it will point to the exact file and line.

- [ ] **Step 4: Start dev server and visually verify**

```bash
npm run dev
```

Check:
- `http://localhost:4321/engineering/` — shows the post in the dense list
- `http://localhost:4321/engineering/groupscout-architecture/` — renders the post with indigo-accented title, source attribution badge, dark code blocks
- Nav "Engineering" link highlights as active on both pages

- [ ] **Step 5: Commit**

```bash
git add src/content/engineering/groupscout-architecture.md
git rm src/content/engineering/.gitkeep 2>/dev/null || true
git commit -m "feat(content): add sample engineering post; verify build passes"
```
