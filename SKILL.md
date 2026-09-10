---
name: custom-css
description: Helps identify the correct, stable CSS class/selector for customizing a Stonly Knowledge Base, and audits existing custom CSS (including AI-generated CSS) for fragile dynamic/hash class selectors that will break on the next deploy.
disable-model-invocation: false
---

# Custom CSS for Knowledge Base

## Purpose

A customer wants to visually customize their Stonly Knowledge Base (colors, spacing, fonts,
hiding/showing elements, etc.) using the KB's custom CSS field. This skill helps identify the
correct class/selector for the element they're describing, and writes CSS that won't break on
KB updates.

This does not rely on a single maintained list of class names — Stonly ships new components over
time and a hardcoded list would go stale. Instead it checks a small set of sources in priority
order (see below), falling back to a classification rule that works on whatever classes are
actually present.

## Sources, in priority order

1. **The public custom-CSS guide** — `https://stonly.com/kb/guide/en/stonly-custom-css-guide-eyAnqX3I5g`
   (English is the only authored locale — don't substitute other locale prefixes like `/pl/`, they
   just redirect back to `/en/`). It's a public page, so fetch it directly rather
   than asking the customer to paste anything. If the requested element already has a documented
   working example there, use that selector as-is — it's a maintained, verified answer, not an
   inference, and takes priority over everything below.

   **How to actually read it — do not just `WebFetch` the URL and trust the summary.** This is a
   multi-step Stonly guide (it's dogfooding the product it documents), not a flat page:
   - A plain `WebFetch` of the bare guide URL only renders the first/landing step — a table of
     contents with no selectors on it. Confirmed empirically: the real per-element content (search
     bar, header, logo, etc.) lives on other steps.
   - `WebFetch`'s LLM-based summarization on this page has been observed to both **hallucinate**
     plausible-sounding prose that isn't actually on the page, and to **miss real content** that
     lives outside the field it expects. Don't rely on it for this guide — verify with a raw fetch.
   - **Correct method:** fetch the raw HTML with `curl` (Bash tool), using a real browser
     User-Agent and following redirects, e.g.
     `curl -sL -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "https://stonly.com/kb/guide/en/stonly-custom-css-guide-eyAnqX3I5g"`
     — no need to know a specific step ID or build a `/Steps/<id>,<id>,...` chain first; a single
     fetch of the base URL embeds the **entire guide's step tree and content** in one response.
   - **Cache the fetch — don't re-`curl` for every question in the same conversation.** The fetch +
     `jq` parse of the ~330KB payload is the slow part of this whole skill, not the lookup itself,
     and the guide's content doesn't change minute-to-minute. After the first fetch+sanitize (see
     the `jq`/`sed` steps below), save the sanitized JSON to a scratch file (e.g.
     `/tmp/stonly-css-guide-cache.json`) next to a timestamp. On a later question in the same
     conversation, check that file's age first — if it's under ~2 hours old, `jq` straight against
     it instead of re-`curl`ing and re-sanitizing from scratch. Only re-fetch from the network if
     the cache is missing, stale, or doesn't contain the step you're looking for (which could mean
     the guide grew since the cache was made).
   - **This isn't unique to the base URL — every URL for this guide returns the same full blob.**
     Confirmed empirically (2026-09-02): fetching a single specific step bare (e.g.
     `.../Steps/5925389`, no ancestor chain) returned the same ~330KB payload, with every step's
     title and content, as fetching the root. This is standard SPA hydration (the whole app state
     ships so client-side navigation needs no more round trips) and will keep holding as the guide
     grows — there is no smaller/scoped URL to jump to, so don't bother hardcoding a `stepId`/path
     lookup to "save" a fetch; it wouldn't reduce what comes back.
   - **Because of that, extract — never bulk-load.** Once fetched, pull out only the matched step's
     `stepModule`/`media` fields (e.g. with `jq`/`grep`/a small script) and work from that. Do not
     `Read` the raw saved HTML/JSON file wholesale or paste the full blob into context — it holds
     every step's content regardless of which one you asked about, and only grows as the guide
     does, so reading it in full defeats the point of matching a specific step by title.
   - **The blob isn't valid JSON as-is — `jq .` fails outright on it.** Confirmed: it contains
     bareword `undefined` (a JS-object-literal token, not JSON — seen ~40 times in one fetch, e.g.
     `"showCompletedSteps":undefined`), which breaks both `jq` and `JSON.parse`/`json.loads`.
     Sanitize before parsing — `sed 's/:undefined/:null/g'` on the extracted blob is enough — then
     `jq` works normally, e.g. end-to-end for one step by title:
     `jq '.initialData.loadedGuide.steps[] | select(.stepModule[]?.content.en.title == "Tiles") | {stepId, content: [.stepModule[].content.en], media}' state_clean.json`
     (verified 2026-09-02: this took a ~330KB fetch down to one step's actual content, well under
     2KB). If you'd rather not parse JSON at all, grep with enough context around the matched
     `"title":"..."` occurrence works too — just don't skip straight to `Read`ing or catting the
     whole file because `jq .` errored.
   - In that HTML, find the `window.__SERVER_APP_STATE__ = {...};` script blob and read its
     `steps` array. Each entry has a `stepId` and `title` — match the target element to a step by
     title (current titles include: "Search", "Header", "Footer", "Logo", "Layout",
     "Contact us", "Featured guides", "Favorites", "Breadcrumbs", "Custom header & footer",
     "Widget customisation", "Language switcher", "Mobile menu", "Structuring your code" — this
     list will grow as the guide is written, don't hardcode it).
   - Each step's real content lives in **two separate fields — check both**:
     - `stepModule[].content.en.content` — narrative/explanatory HTML text.
     - `media.en[].content` (only present when `mediaType` is `"code"`) — the actual example CSS.
       **This is usually where the real selector/example is** — don't stop at the narrative text
       and conclude a step has "no documented selector" without checking `media` too.
   - **Don't trust the `media.en[].mode` label to decide whether a step has CSS.** It's meant to be
     `"css"` but has been seen mislabeled `"javascript"` on a step (Logo) whose `content` was
     actually a CSS block followed by a JS snippet, both real. Read `media.content` whenever
     `mediaType` is `"code"`, regardless of what `mode` says — don't filter/skip based on `mode`.
   - **`mediaType: "image"`** (seen on "Custom header & footer", "Structuring your code") means the
     step's content is a screenshot, not text — there's no selector to extract. Treat that exactly
     like a step with no documented example (fall through to source 2/3), not as a crawl failure.
   - A step whose `media` is absent/empty and whose `stepModule` content is `""` genuinely has no
     content yet (the guide is a draft, being written incrementally) — that's a real "not
     documented yet," not a crawl failure. Don't mistake the two: if you got here via `WebFetch`
     instead of raw `curl`, re-verify with `curl` before concluding a step is empty. (Empty content
     is currently normal on the landing/table-of-contents steps — "Customize your Knowledge Base
     and Guides with CSS", "Knowledge Base customization", "Widget customisation", "Custom css AI
     skill" — those are navigation menus, not element documentation.)
2. **`reference/ston-classes.md`** (in this skill folder) — a point-in-time snapshot of known
   `ston-*` classes grouped by area. Won't have anything shipped after its snapshot date, but
   nothing in it goes stale/wrong either.
3. **Classification rules** (below), applied to a live class list obtained via DevTools or a
   screenshot — the fallback when the element isn't covered by either source above.

**Coverage in any of these sources is a positive signal only, never a negative one.** The guide in
particular is a work in progress — a class missing from it just means "not documented yet," not
"suspect." Never flag or downgrade a class merely because it isn't in the guide or the reference
snapshot; only the hash-pattern rules in Classification rules are grounds to flag a class as
fragile. This applies to both workflows below, including the audit workflow.

## Two entry points

This skill is used two ways — figure out which one applies before doing anything else:

- **Picking a selector** — someone wants to style an element they haven't targeted yet. Go to
  "Workflow: picking a new selector".
- **Auditing existing CSS** — someone pastes CSS they (or an AI) already wrote, usually because
  something "stopped working" or they want a sanity check before saving it. Go to "Workflow:
  auditing existing CSS". Default to auditing whenever CSS is pasted in, even if not explicitly
  asked — catching a fragile selector before it ships is the main point of this skill.

## Response format

Default to the shortest response that gets the customer to a working answer:

- Lead with the CSS block. Don't narrate the verification process ("I checked the guide, confirmed
  the class is stable...") — that reasoning is for you to have done, not for the customer to read.
- After the CSS, at most one short follow-up sentence (e.g. offering to adjust the exact
  shade/value). Don't lay out every alternate selector variant (section vs. per-item vs.
  single-instance) as a menu in the same response — if there's a genuine fork between two
  documented targets, resolve it per "Resolving 'each item' vs. 'the whole section' ambiguity"
  below *before* writing any CSS, rather than answering one and listing the rest as alternatives.
- Only go longer than CSS-plus-one-sentence when something needs flagging per the audit workflow
  (fragile selector, a Known Gotcha, unnecessary `!important`, etc.) — and even then, keep the flag
  itself to a sentence or two, not a full explanation of the classification rules behind it.

### Resolving "each item" vs. "the whole section" ambiguity

Several KB-home-page elements are documented at two levels at once: a per-item card class and a
separate wrapper/section class around the whole group — confirmed pair: `.featured-article` (each
card) vs. `.featured-articles-section` (the section behind all of them); the `.folder`/`.subfolder`/
`.guide` family under "Scoping to one specific card/block" below likely splits the same way. A
request phrased around the plural/group name ("featured guides", "the folders section") doesn't
disambiguate between these on its own — "make X red" is an equally normal way to ask for either
one.

This is a different situation from the general ambiguous-request handling (answer the confident
interpretation, flag the uncertain one, don't block with a question) — that rule is for when one
interpretation is source-confirmed and the other is a lower-confidence guess. Here, both
interpretations are equally well-documented and equally plausible, so there's no confident default
to lead with. **Ask which one the customer means before writing any CSS**, rather than guessing one
and listing the other as a footnote — a wrong guess means they paste it, see the wrong thing turn
red, and have to come back for a second round.

## Workflow: picking a new selector

1. Ask which element they want to change, in their own words (e.g. "the search bar", "article
   titles", "the sidebar navigation", "the feedback buttons at the bottom of an article"). If the
   request names a group of repeating items (guides, folders, featured articles, etc.) without
   saying "each"/"every" or "the section"/"the area", resolve the item-vs-section ambiguity above
   before going further.
2. Check the Sources above in order. Only drop to DevTools/screenshot-based classification once the
   guide and the offline reference have both come up empty for this element. If neither the guide
   nor DevTools/a screenshot is available, ask for one rather than guessing — a wrong selector
   fails silently (matches nothing, applies no style), which is worse than asking one more question.
3. If working from a live class list, classify each class using the rules below and pick the most
   stable one.
4. If the resulting selector isn't guaranteed unique on the page, scope it (see Scoping below).
5. Write scoped, minimal CSS — target the specific class, avoid `!important` unless something is
   already overriding it, and avoid broad selectors (`div`, `*`) that could affect unrelated
   elements.

## Workflow: auditing existing CSS

This is the higher-leverage case — customers and AI both tend to reach for whatever selector is
in front of them, including dynamic ones, and it works until the next deploy regenerates the
hashes. Given a block of custom CSS:

1. Extract every selector and classify each class in it using the rules below.
2. Flag as **fragile** any selector containing:
   - a class that matches the rule-4 hash patterns, or
   - a rule-3 styled-components class used **literally, with its `-sc-<hash>-<N>` suffix intact**
     (e.g. `.Home-styles__HeaderBackgroundWrap-sc-20c6f379-6`) — "semi-stable" describes the
     `ComponentName__Thing` prefix once isolated, not the exact class as originally written; used
     as-is it depends on that hash suffix matching exactly, which regenerates on rebuild just like
     a rule-4 class.
   In both cases, explain that it will silently stop matching after the KB's next build, and that
   this is a common cause of "my custom CSS broke for no reason" reports.
3. For each flagged selector, propose a stable replacement, **checking in the same rule-1/2/3/4
   priority order as Classification rules — a `ston-*` or static/semantic sibling always wins over
   a styled-components rewrite, even when a styled-components class is also sitting right there:**
   - First, check whether a `ston-*` (rule 1) or static/semantic (rule 2) class is present on the
     same element — if so, that's the answer, full stop. Don't reach for a rule-3
     styled-components rewrite just because one is also present alongside it.
   - Otherwise, if a rule-3 styled-components class (meaningful prefix + hash) is present — whether
     that's the flagged selector itself, or sitting alongside a flagged rule-4 hash on the same
     element (the common case: `sc-<hash>`/short companion hashes like `hkEXdL` are usually copied
     from the same live element's full class list as their governing styled-components class, e.g.
     `sc-bZQynM`/`hkEXdL` alongside `Home-styles__CenteredTitle-sc-d1f1c749-1`) — rewrite that class
     as `[class*="ComponentName__Thing"]`, discarding the hash suffix. This is a mechanical, safe
     rewrite; no need to ask the customer for anything.
   - Only if neither a rule-1/2 class nor a rule-3 class is present anywhere on the element, say so
     explicitly and ask for the element's live class list rather than guessing a replacement.
4. Also flag rules that are technically stable but too broad — on **either axis**, same failure
   mode, different cause:
   - Broad **selector**: bare tag selectors, `*`, unscoped generic classes likely reused elsewhere.
   - Broad **property**: `all: unset` / `all: initial` / `all: revert` on a narrow, correctly-scoped
     selector is just as much a "too broad" problem even though the selector itself is fine — it
     wipes every property on the element, not just the ones the customer meant to override. On a
     form element like `.ston-search-input`, that silently strips things they almost certainly
     didn't intend to touch too — the focus outline (an accessibility regression), cursor, font
     inheritance — not just the background/border they were trying to reset. Flag it and suggest
     resetting only the specific properties actually being overridden instead.
5. **A selector can be fully stable, correctly scoped, and pass steps 2–4 clean, while the CSS
   still silently does nothing. Check the pasted CSS against every bullet in Known Gotchas below —
   don't stop at classifying each selector in isolation, and don't limit the check to the shapes
   illustrated here.** The examples below are different *shapes* this failure mode takes, not an
   exhaustive list of which gotchas matter — treat every bullet in Known Gotchas as in-scope for
   this step, including ones with no worked example here yet:
   - **A Stonly-specific reason a selector won't apply no matter how correctly it's written.** Not
     hypothetical: `.guide-YbHBPIxT1u .ston-progress-bar-bar { ... }` uses a stable `ston-*` class
     and the documented guide-scoping prefix correctly — a textbook-correct answer by every other
     measure — but per "Progress bar CSS is global-only," that scoping silently doesn't apply.
   - **Rules against each other** — a plain CSS cascade conflict entirely within the pasted block,
     nothing Stonly-specific about it, the "Shorthand vs. longhand conflicts" gotcha applying to
     the customer's own rules. Also not hypothetical: pasted CSS containing both
     `.featured-articles-section { background-color: red; }` and, later, `.featured-articles-section, .recent-guides-section { background: #F6F1EB; ...}`
     — the later shorthand rule resets `background-color` back to beige at equal specificity.
     Whenever the pasted CSS has more than one rule, check whether any pair targets overlapping
     properties on the same (or overlapping) selector — don't just classify each rule's selector
     in isolation and move on.
   - **The right property for what the element actually is, not just a stable selector for it** —
     the "SVG icons: fill vs. stroke" gotcha. `.ston-favorite-content-loader { fill: #4f46e5; }`
     uses a perfectly stable class, but the loader is stroke-based (confirmed live: `.ston-favorite-content-loader circle { stroke: ... }`)
     — `fill` silently does nothing on it, and the wrapper isn't even the right target, the `circle`
     child is. A stable class doesn't mean the property or the exact target element is right.
   - **Deployment-context gotchas the CSS can't reveal on its own** — e.g. "Embedded guides /
     Guided AI Answers ignore their own CSS" depends on *how the guide is currently being shown*,
     not on anything in the CSS. A `.guide-<ID> .ston-content-text { ... }` rule using a perfectly
     stable class and correct scoping can still be exactly this case, with no way to tell from the
     selector alone. When the customer's report matches this shape — "it used to work and
     stopped" with no dynamic classes in sight, or "this is correct but isn't applying" with
     nothing else wrong — don't conclude the CSS is simply fine; ask the deployment-context
     question the relevant gotcha depends on instead of only checking what the CSS can show you.
   Whichever bullet applies (including ones not shown above), flag it as its own category (not
   "fragile," not "too broad" — just "won't work," or "might not work, need to check X") and
   explain specifically why, rather than letting it pass because it satisfied every other check.
6. **Flag unnecessary `!important`, even on otherwise-fully-correct selectors.** This isn't a
   "won't work" case like step 5 — it's a maintainability smell worth calling out on its own, per
   "`!important` is only needed when something else already wins" in Known Gotchas. Blanket
   `!important` across most/all rules in a pasted block (common in AI-generated CSS, which tends to
   add it defensively) makes future overrides harder without fixing anything. Ask whether it was
   added because something else was actually overriding the style, or just as a defensive habit —
   don't silently pass it through just because the selectors underneath it are otherwise stable.
7. Leave selectors that already use `ston-*`/semantic classes alone — don't rewrite CSS that's
   already correct **and confirmed not to hit any case in steps 5–6.**

## Classification rules

Given the raw class list on an element, classify each class, then **pick in this priority
order — fully-stable classes always outrank a styled-components attribute selector, even
though a styled-components class is still usable as a last resort:**

1. **`ston-*` prefix → stable, always preferred.** This is Stonly's intentional public styling
   API (e.g. `ston-kb-home`, `ston-field-check`). Use it directly as `.ston-kb-home`.

2. **Everything else with no hash pattern → semantic, stable — equally preferred to `ston-*`.**
   Clean class names — `kb-*` prefixes, BEM-style names, plain utility names (e.g.
   `kb-header-title`, `contact-us`). Treat these exactly like `ston-*`: usable directly as
   `.kb-header-title`. **If a class like this exists on the element, use it — don't reach for a
   rule-3 styled-components attribute selector instead just because it's also present.** A static
   class needs no workaround and no hash to discard; a styled-components class always does.

3. **Styled-components with a meaningful prefix → semi-stable, use as attribute selector, and
   only when neither rule 1 nor rule 2 gives you anything on this element.** Pattern:
   `ComponentName__Thing-sc-<hash>-<N>` (e.g. `Home-styles__HeaderBackgroundWrap-sc-20c6f379-6`).
   The `ComponentName__Thing` part is meaningful and reasonably stable; the `-sc-<hash>-<N>` suffix
   is not. Use `[class*="ComponentName__Thing"]`, discarding the hash suffix. Confirmed real-world
   case: an element with classes `Home-styles__CenteredTitle-sc-d1f1c749-1
   Home__HomeTitle-sc-a12f3f67-2 bzTuEc bgclVW kb-header-title` — don't rewrite either
   styled-components class here; `kb-header-title` (rule 2) is already stable and sitting right
   there. The two short hashes (`bzTuEc`, `bgclVW`) are rule-4 companion hashes of the two
   styled-components classes — never usable regardless.

4. **Pure hash classes → discard, do not use for styling.** These change on every rebuild:

   - `sc-<hash>` alone (e.g. `sc-20c6f379`)
   - Short 5-8 char no-separator strings on an element that has a styled-components sibling class
     (e.g. `hkEXdL`, `gvttqa`) — these are the styled-components "companion hash"
   - Standalone mixed-case single-dash hashes (e.g. `gA-dTyl`)

If the element has no usable class of its own, don't fall back to guessing a selector — walk up
to the nearest ancestor that has a `ston-*` or semantic class and build a path from there (e.g.
`.ston-content-text ol li p`), preferring semantic HTML tags (`ol`, `li`, `nav`, `table`, `header`,
etc.) in the path over generic `div`/`span`.

**Not just classes — two other selector types are equally legitimate when confirmed live:**

- **Plain `id` attributes on singleton elements** (e.g. `#language-switcher`,
  `#listbox-for-language-switcher`) — as stable as a semantic class; use `#the-id` directly.
- **ARIA role/state attributes** (e.g. `[role="option"][aria-selected="true"]`,
  `[aria-expanded="true"]`) — required for accessibility, so Stonly is unlikely to rename or drop
  them; safe to use as a stable selector, including combined with a class for extra precision.

## Scoping / uniqueness

A selector that looks stable can still match more than one element on the page (e.g. a generic
`kb-header-title` class reused across multiple widgets). Before finalizing:

- If you can't verify uniqueness (no live page access), say so and suggest the customer verify by
  checking how many elements the selector matches, rather than asserting it's unique.
- If scoping is needed, prefix with the nearest stable ancestor: `.ston-kb-home .kb-header-title`,
  preferring a `ston-*` ancestor over a semantic one, over an attribute-selector prefix.

### Scoping to one specific guide or step

Separate from ancestor scoping above: Stonly provides two dedicated scoping prefixes rather than
needing to build a more specific ancestor-class selector for these cases:

- `.guide-[GUIDE_ID]` — restricts a rule to a single guide, e.g. `.guide-YbHBPIxT1u
.ston-content-text { ... }`. The guide ID comes from its editor URL —
  `https://app.stonly.com/app/guide/<GUIDE_ID>/editor/...` — the segment right after `/app/guide/`.
- `.step-[STEP_ID]` — restricts a rule to a single step, e.g. `.step-213700 .button-wrap
.back-button { display: none; }`. Use this when the customer says "just this one step," not the
  whole guide.

Use whichever matches what the customer actually asked for ("just this one guide" vs. "only this
step") — these are the mechanisms Stonly actually provides for it, not something to approximate
with ancestor classes. The step wrapper also carries the same ID as a `data-step-id="<STEP_ID>"`
attribute (confirmed live), so `[data-step-id="213700"] .button-wrap .back-button` is an equivalent
alternative to the `.step-[STEP_ID]` class form if you ever need the attribute form specifically —
same underlying ID, just exposed two ways, same pattern family as the card-level scoping below.

### Scoping to one specific card/block (folder, guide, subfolder, featured article)

A related but distinct case from the guide/step scoping above: several KB-home-page blocks that
repeat per-item (folder cards, subfolder sections, guide/article cards, featured-article cards)
carry a `data-*-id` attribute alongside their class, letting you target **one specific instance**
without touching the others — this scopes one card *on* the home page, not a whole guide/step's
content. Confirmed live in the guide's own examples:

```css
.folder[data-folder-id="469"] { background: red; }
.subfolder[data-folder-id="2063"] { background: red; }
.guide[data-guide-id="S0rfooZx37"] { background: red; }
.featured-article[data-guide-id="123456"] { color: #4f46e5; }
```

The ID is visible in the app URL when viewing that folder/guide. Use this when the customer wants
to style "just this one folder/article," not the whole family of cards — don't reach for the
`.guide-[GUIDE_ID]` prefix above for this, that's a different mechanism (scopes a whole guide's
*content* pages, not one card *on* the home page).

Note `.folder`/`.subfolder`/`.guide`/`.featured-article*` are plain semantic classes, not `ston-*`
— per Classification rule 2 they're just as stable, but being non-`ston-*` they won't show up in
`reference/ston-classes.md` even after a refresh (that file is generated from Stonly's frontend
source, which can only find literal `ston-` classes). The guide is the source of truth for this
pattern, not the reference snapshot.

## Known gotchas

- **`!important` is only needed when something else already wins.** Don't add it by default —
  only when the customer reports the style isn't taking effect, which usually means a more
  specific existing rule is overriding it.
- **A documented `ston-*` class can be outranked by Stonly's own internal CSS, not just by the
  customer's other rules.** Some `ston-*` classes lose to a Stonly-internal styled-components
  selector purely on specificity (attribute selector + class beats a single class) — this happens
  in a KB with zero other custom CSS, so it's not a customer authoring mistake and won't show up
  from reading the pasted CSS alone. Confirmed case: `.ston-folder-block-name`'s `color` loses to
  `.folder [class*="common-styles__BlockName"]` — see `reference/ston-classes.md`'s "Known
  specificity conflicts" table for the full list and how each was confirmed. This skill has no
  browser access, so it can't detect a new instance of this live — if a customer reports a stable,
  correctly-scoped selector "just not applying" with no other custom CSS to blame, this is a
  candidate explanation; check the table above first, and if the class isn't listed, suggest they
  check via browser DevTools (Elements → Styles panel — an overridden property shows struck
  through) rather than asserting the selector must be fine. When confirmed, `!important` on just
  the affected property is the correct fix per the bullet above, not a broader rewrite.
- **Shorthand vs. longhand conflicts.** e.g. a `grid-template` shorthand rule elsewhere will
  silently override a longhand `grid-template-columns` override. If a change "does nothing,"
  check whether a shorthand property is the real target.
- **SVG icons: fill vs. stroke.** Some icons are stroke-based (`fill: none`, color carried by
  `stroke`) — setting `fill` does nothing on these. Check which attribute actually carries the
  color before writing the override, and never use `color` on SVG/path elements directly.
- **Widget/iframe embeds.** If the KB is embedded via iframe on the customer's own site, the
  selector still needs to target elements _inside_ the KB's own stylesheet/custom CSS field — it
  can't reach into the iframe from the parent page's CSS.
- **Embedded guides / Guided AI Answers ignore their own CSS.** When a guide is embedded inside
  another guide, or loaded as a Guided AI Answer, its own custom CSS (including any `.guide-<ID>`
  scoped rules) is not applied — the parent guide's CSS applies instead. If a customer says their
  per-guide CSS "works standalone but not here," check whether that guide is actually being shown
  embedded/as a Guided AI Answer before assuming the selector is wrong.
- **Progress bar CSS is global-only.** It cannot be scoped to one guide or step — only a site-wide
  rule (no `.guide-<ID>`/`.step-<ID>` prefix) actually applies. If a customer asks to restyle the
  progress bar for just one guide, say so explicitly rather than writing a scoped selector that
  will silently do nothing.
- **Not every real class is `ston-*`.** Plenty of stable, currently-working selectors in Stonly's
  own documentation are plain semantic classes (e.g. `.content-text`, `.radio-wrap`,
  `.checklist-icon`, `aside.tip`/`aside.warning`) or styled-components attribute selectors (e.g.
  `[class*="ProgressBar__Bar"]`, `[class*="Npsstyles__Label"]`), not `ston-*`. The Classification
  rules already treat clean non-hash classes as stable (rule 2) — don't second-guess a selector
  just because it lacks a `ston-` prefix.
- **Dynamic/hash classes look plausible but aren't safe.** If a class matches the hash patterns in
  the Classification rules above, don't use it even if it happens to work right now — it will
  break on the next deploy.
- **Positional selectors (`:nth-child`, `:first-child`, `:nth-of-type`, etc.) target a position,
  not a specific item — a different, non-hash-related kind of fragility.** `.ston-folder-grid >
  a:nth-child(1)` doesn't mean "the featured folder," it means "whatever currently renders first."
  It silently starts matching a *different* folder the moment someone reorders folders, adds a new
  one, or archives one — no error, no visual warning, just the wrong content highlighted. Unlike a
  hash class, this doesn't break on a KB rebuild — it breaks on ordinary content editing, which
  makes it easy to miss in a CSS review that only checks for dynamic classes. If the goal is to
  style one *specific* item regardless of its position, use the `data-*-id` attribute pattern from
  "Scoping to one specific card/block" above (e.g. `.folder[data-folder-id="469"]`) instead of a
  positional selector — that's what it exists for.
- **The same class can be applied to two nested elements at once, not just to sibling variants.**
  Confirmed example: in the Favorites section, `.ston-favorite-content-title` is applied both to the
  entire clickable row (an `<a>`) and, inside it, to a smaller wrapper around just the icon + title
  text. A rule on `.ston-favorite-content-title` therefore matches both elements simultaneously —
  any `display`/`padding`/`border` set on it applies twice, nested. If only one of the two is
  wanted, target a distinguishing child instead (e.g. `.ston-favorite-content-title-text`, or a
  child combinator off a sibling like `.ston-favorite-content-icon`). Also note this same section
  exposes its loading/empty/populated states as modifier classes alongside the base class
  (`.ston-favorite-content-section.loading` / `.empty` / plain) — see `reference/ston-classes.md`'s
  "Favorites section structure" note for the full worked example.
- **A shared name prefix doesn't mean a shared DOM element.** Classes that look like modifier
  suffixes of one base class (e.g. `ston-kb-header-fixed`, `ston-kb-header-sticky`) can actually be
  entirely separate elements rendered in different states, each needing its own rule — not one
  selector covering all of them. Confirmed example: `.ston-kb-header` (the main header) and
  `.ston-kb-header-sticky` (the scrolled/pinned header) are two different DOM subtrees with their
  own independent `background-color`; styling only `.ston-kb-header` leaves the sticky header
  showing the old color once the visitor scrolls. Before assuming a single class covers "the
  header" in every state, verify whether the suffixed variants are modifiers on the same element or
  separate elements — see `reference/ston-classes.md`'s "KB header structure" note for a worked
  example, and treat other `*-fixed`/`*-sticky`/state-suffixed class families with the same
  suspicion until checked.
- **Check for a built-in KB setting before writing layout/visibility CSS.** Not just hide/show —
  some layout changes customers ask for in CSS terms (column counts, grid vs. list) are actually a
  first-class setting too. All under **Knowledge base tab → Settings → Layout**:
  - **Folder/guide grid column count** — the "Content display" dropdown (one per folder depth
    level) offers **Tiles - 2/3/4 columns** and **List - 1/2 columns**. A request like "folders
    should be 2 columns instead of 4" is this setting, not a custom-CSS case — suggest it first,
    and only reach for a `grid-template-columns` override if the customer specifically wants a
    column count the dropdown doesn't offer (e.g. more than 4).
  - **Contact us (home page section only)** — under the Contact block, "Display options" has a
    **"Button inside Knowledge base"** checkbox. Unchecking it hides just the home-page
    section/button while leaving the contact form active in other placements (widget, KB header,
    guide header, search results) if those are separately checked. This is the right toggle for
    "hide the Contact us section on the home page" — don't reach for `display: none`.
    - The broader "Contact type" dropdown also has a **None** option, but that disables the
      contact form everywhere, not just the home page — only use it if the customer wants contact
      removed entirely.
  - **Recent guides** — the "Update type" dropdown has a **None** option that disables the whole
    block (confirmed live).
  - **Favorites** — a dedicated **Yes/No** enable toggle (confirmed live).
  - **Featured articles** and the folder/guide grid itself have **no visibility toggle** (column
    *count* is configurable per the "Content display" dropdown above, but hiding the grid entirely
    is not) — the featured block simply disappears once every featured article is removed via the
    same panel (it's driven purely by whether any exist, not a separate toggle), and the main grid
    always renders. `display: none` is legitimate for hiding these if a customer genuinely wants
    that via CSS.
  Before writing a hide rule *or* a layout override for any KB-home-page block, check which case it
  falls into rather than defaulting to CSS.
