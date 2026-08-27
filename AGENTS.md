# Documentation project instructions

Public documentation for Impala AI, built on [Mintlify](https://mintlify.com). Pages are MDX with
YAML frontmatter; navigation and theme live in `docs.json`.

- `mint dev` — local preview on :3000
- `mint broken-links` — check internal links
- `mint validate` — validate the build

For Mintlify product knowledge (components, configuration, schema), prefer the live docs at
[mintlify.com/docs](https://mintlify.com/docs) over anything you remember.

## Before you write

1. Read `docs.json`. It defines every tab, group and page, and the naming conventions in use.
2. Read 2–3 neighbouring pages and match their register and depth, not just their formatting.
3. Check whether the content belongs on a page that already exists. Adding a section beats adding
   a page; linking beats restating.

## Voice

These docs are written for a **qualified reader** — someone who already knows what inference is and
is evaluating or integrating. Don't warm up a cold audience. Don't sell.

### Get to the point

- **No framing prose.** Cut openers that explain what the page is about ("this guide shows how
  to…", "the docs explain what X does, these explain why") and closing paragraphs that restate it.
- **No signposting.** "Three things to do before anything else on this page" is noise; the list
  speaks for itself.
- **One page, one job.** If a page makes the same argument in prose, then a table, then a diagram,
  then an aphorism, keep whichever carries the most information and cut the rest.
- **Short is the default.** An orientation page past ~400 words is usually repeating itself.

### Structure has to earn its place

Prose is the default. Add a heading, table, callout or diagram only when it carries something
prose can't.

- A **table** is reference material a reader scans and returns to. A table restating the paragraph
  above it is rhetoric — cut it.
- A **diagram** must show a mechanism. If one sentence already makes the point, it's decoration.
- **Three headings over 200 words** means the headings are doing nothing.

### Titles and descriptions

- Page titles **name the subject**. They are not claims. "Run in our cloud, or yours" is a title;
  "Cheaper inference, better quality" is a claim and belongs in `description`.
- `description` renders as the subtitle under the H1. Write it to read as one.
- Keep case **consistent within a nav group** — pick sentence case or title case and hold it.
- Peer pages in a group should share construction, so the sidebar has a rhythm.

### Language

- Sentence case for headings and code block titles. Second person. Active voice.
- Bold is for **defined terms on first mention**, not for decoration and not mechanically at the
  start of every bullet.
- One em-dash per paragraph at most.
- Avoid: powerful, seamless, robust, cutting-edge, simply, just, easily, obviously, leverage,
  unlock (as a verb), "it's important to note", "in order to".
- Avoid negative parallelism ("not X, but Y"), three-item lists used for cadence rather than
  because there are three things, one-sentence paragraphs for drama, and closing flourishes.

### When you're unsure of a fact

Be conservative with wording and with how many concepts you introduce. A confident framing that
turns out to be wrong is worse than a plain one — it makes the page unusable rather than merely
thin. Mark anything unconfirmed:

```mdx
{/* TODO: confirm the default timeout */}
```

Don't invent parameters, limits, or error shapes. If the OpenAPI spec doesn't cover it, say so in
a TODO rather than filling the gap.

## Conventions

- Kebab-case filenames: `getting-started.mdx`.
- Internal links are root-relative with no extension: `/send-a-request`. Never `../` and never an
  absolute `https://docs.getimpala.ai/...` URL for an internal page.
- Every page needs `title`; include `description`.
- Every code block needs a language tag. Use realistic values, never `foo`/`bar`.
- Every image needs descriptive alt text. Store images in the repo root or `/images`; no spaces in
  filenames.
- A new page must be added to `docs.json` or it won't appear in the sidebar.
- `platform.getimpala.ai` is the customer platform — sign-up, endpoints, API keys.
- State what is supported. Don't enumerate what isn't.

## Gotchas found the hard way

1. **`{"anchor": …, "href": …}` inside a group's `pages` array is invalid and silently breaks the
   entire tab** — unrelated pages start rendering the wrong sidebar. Only page-path strings and
   group objects belong in `pages`. To link out, use `navbar.links`.
2. **MDX strips `<text>` nodes from inline SVG.** Labelled diagrams have to be HTML/CSS; flex divs
   with percentage widths render reliably.
3. **Mermaid `gantt` with `dateFormat X`** reads `:start, end` as `:start, duration`, so every bar
   starts at zero. Don't use gantt for timing diagrams.
4. **`colors.light` is used in dark mode and `colors.dark` in light mode.** They're contrast
   partners, not mode names.
5. **A page missing from `docs.json` is hidden, not broken** — still reachable by URL and still
   indexed for search and the docs assistant. Fine for drafts, as long as that's intended.
6. Branch previews rebuild in roughly 60–120 seconds. **Verify a commit by reading the committed
   file, not the preview.**

## Before you open a PR

- [ ] `title` and `description` present; description reads as a subtitle
- [ ] New pages added to `docs.json`
- [ ] Internal links root-relative, no extension
- [ ] Code blocks tagged; values realistic
- [ ] No marketing words, no framing prose, no closing flourish
- [ ] Case consistent with the rest of the nav group
- [ ] No claim contradicted by another page — search for the claim before shipping it
- [ ] TODOs on anything unconfirmed
- [ ] `mint broken-links` and `mint validate` pass
