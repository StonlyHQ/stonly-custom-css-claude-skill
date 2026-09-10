# Known `ston-*` classes

A known-good snapshot of Stonly's public `ston-*` styling API. These are intentionally exposed for
customer custom CSS — this is a documented, stable customization surface, not an implementation
detail.

**This list will go stale, and that's fine** — it doesn't need to be kept in sync. A class here is
still correct even years later; it just won't include anything shipped after this snapshot. When a
class isn't found here, fall back to the classification rules in `SKILL.md` rather than assuming
it doesn't exist.

## Known issues (don't recommend these as-is)

- **`ston-fieldGroup-main`** (camelCase) vs **`ston-fieldgroup-main`** (lowercase) — two
  differently-cased versions of what looks like the same intended class, likely drift between two
  parts of the product. Use whichever one you actually observe on the live element rather than
  assuming.
- **`ston-statusIcon`** (camelCase) vs **`ston-status-icon`** (kebab-case) — same drift pattern.
- **`ston-code-block` is two different things — the 2026-08-31 "does not exist" correction was only
  half right, fixed 2026-09-07.** The customer's own typed rich-text code block is still correctly
  classless — confirmed live: it's raw `<pre data-type="code-block">`/`<code data-type="code-block">`
  markup, use the attribute selector `pre[data-type="code-block"]` for it. But a **separate "code"
  illustration/media-pane type** (a rendered code snapshot with `added`/`removed`/`modified` diff
  styling, confirmed live) genuinely renders `className="ston-code-block"` on its root — real and
  currently shipping. See the "Article / guide player" table below for both.
  `ston-copy-code-button` (the hover-visible copy button on the rich-text version) is real and
  unaffected either way.
- **`ston-step-buttons` is an `id`, not a class, and only covers the overflow button.** Corrected
  2026-09-02 after live verification: it's applied as `id="ston-step-buttons"` on the **"More
  actions" overflow trigger** — the "…" button shown only when there are more step-footer action
  buttons (feedback, favourites, TOC, etc.) than fit — not on the row itself. As an id it needs
  `#ston-step-buttons`, not `.ston-step-buttons`. The row's actual wrapper class is
  `stepBottomButtons` (plain, confirmed real, not `ston-*`).
- **`ston-feedback-title` is also an `id`, not a class — same pattern as `ston-step-buttons`
  above.** Confirmed 2026-09-07 via live inspection. A compact `ston-feedback-button, -title`
  notation might read as if `.ston-feedback-title` were a class — it isn't; needs
  `#ston-feedback-title`.

## Known specificity conflicts (ston-* overridden by Stonly's own internal CSS)

Confirmed cases where a documented `ston-*` class loses to a Stonly-internal styled-components
selector purely on specificity — **not a customer CSS conflict**, this happens even in a KB with no
other custom CSS at all. Confirmed by manually inspecting the live element's computed styles in
browser DevTools (Elements → Styles panel — an overridden property shows struck through, with the
winning rule listed above it); this skill has no browser access of its own to detect these live
(see `SKILL.md`'s "ston-* outranked by Stonly's own internal CSS" gotcha), so this list only grows
as findings are manually confirmed and logged here.

| `ston-*` class          | Property tested | Overridden by                                  | Fix                                                                 |
| ------------------------ | ---------------- | ----------------------------------------------- | -------------------------------------------------------------------- |
| `ston-folder-block-name` | `color`           | `.folder [class*="common-styles__BlockName"]` (specificity 0,2,0 vs. the class's 0,1,0) | Add `!important` to the `color` declaration on `.ston-folder-block-name` |
| `ston-progress-bar-bar` / `-line` / `-text` | `background-color`, `opacity`, `top`, `visibility` | `.ston-progress-bar-wrap .ston-progress-bar .ston-progress-bar-bar` etc. (specificity 0,3,0 vs. the single class's 0,1,0) — **only when the KB has "increased contrast" accessibility mode ON**, a customer-togglable setting, not always-on | Add `!important` to the affected property, same as the fix above |

When logging a new one here: note the exact property tested (a class can be overridden on one
property and clean on another) and the overriding selector, so future checks don't have to
re-derive the specificity math.

## Design system — forms, modals, dialogs, tooltips, status

Used across many components (buttons, fields, dialogs) rather than tied to one page.

| Class                                                   | Purpose                                                  |
| ------------------------------------------------------- | -------------------------------------------------------- |
| `ston-button-minimal`                                   | minimal/text-style button variant                        |
| `ston-chip`                                             | generic chip/tag element                                 |
| `ston-dialog-content`                                   | content area inside a shared dialog                      |
| `ston-dialog-main`                                      | root/main dialog container                               |
| `ston-dropdown-trigger`                                 | element that opens a dropdown                            |
| `ston-dropdown-wrap`                                    | wrapper around a dropdown/select field                   |
| `ston-field-additionalAction`                           | button for an extra field action                         |
| `ston-field-check`                                      | checkbox input element                                   |
| `ston-field-check-group`                                | group wrapper for checkbox options                       |
| `ston-field-check-label`                                | label text of a checkbox                                 |
| `ston-field-check-tick`                                 | checkmark icon inside a checkbox                         |
| `ston-field-check-wrap`                                 | wrapper around a checkbox field                          |
| `ston-field-file-name`                                  | filename text of an uploaded file field                  |
| `ston-field-file-placeholder`                           | placeholder text of a file input                         |
| `ston-field-main`                                       | generic form-field container                             |
| `ston-field-placeholder`                                | placeholder styling of a field/select                    |
| `ston-field-toggle`                                     | toggle/switch input field                                |
| `ston-field-wrap`                                       | wrapper around a form field                              |
| `ston-fieldGroup-main` / `ston-fieldgroup-main`         | field-group container (see Known issues)                 |
| `ston-focus-locked`                                     | applied when a focus-trap/lock is active                 |
| `ston-input-label`                                      | label text of a form input                               |
| `ston-input-message`                                    | helper/error message under an input                      |
| `ston-input-required-suffix`                            | "required" marker on an input label                      |
| `ston-input-tooltip`                                    | tooltip attached to an input                             |
| `ston-input-wrap`                                       | wrapper around a form input                              |
| `ston-list-item-standard-title`                         | text label inside a generic list-item row (confirmed in the language-switcher popover) |
| `ston-list-search-input-container`                      | container of a reusable list-search input box            |
| `ston-modal-backdrop`                                   | backdrop overlay behind a modal                          |
| `ston-modal-close`                                      | close button of a modal                                  |
| `ston-modal-container`                                  | outer container of a modal                               |
| `ston-modal-fullscreen`                                 | fullscreen modal variant modifier                        |
| `ston-modal-main`                                       | root modal element                                       |
| `ston-notification`                                     | generic notification/toast element                       |
| `ston-portal`                                           | root element for portaled UI (tooltips/dropdowns)        |
| `ston-radio-group-input-wrap`                           | wrapper of a radio-button input group                    |
| `ston-status-error` / `-info` / `-success` / `-warning` | status message variant modifiers                         |
| `ston-status-icon` / `ston-statusIcon`                  | icon shown alongside a status message (see Known issues) |
| `ston-tick`                                             | generic checkmark/tick icon                              |
| `ston-tooltip-main`                                     | tooltip container                                        |
| `ston-tooltipTrigger`                                   | element that triggers a tooltip                          |
| `ston-textarea-character-counter`                       | character counter under a textarea (see Known issues)    |

## KB home page

| Class                                                                                                                           | Purpose                                 |
| ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `ston-additional-footer`                                                                                                        | custom "additional footer" section      |
| `ston-contact-section` / `-title`                                                                                               | "contact us" section                    |
| `ston-favorite-content-*` (icon, list, loader, remove-button, section, section-title, step-title, title, title-text)            | favorites section                       |
| `ston-featured-articles-section-title`                                                                                          | "featured articles" section title       |
| `ston-folder`, `-block-name`, `-custom-icon`, `-description`, `-description-wrap`, `-grid`, `-icon`, `-icon-wrap`, `-text-wrap` | folder block on KB home                 |
| `ston-footer`                                                                                                                   | page footer                             |
| `ston-header-fixed-logo`                                                                                                        | logo in sticky header — see "KB header structure" below |
| `ston-header-scene`                                                                                                             | header wrapper on home / search results — see "KB header structure" below |
| `ston-kb-container`                                                                                                             | main content container                  |
| `ston-kb-cover-photo`                                                                                                           | modifier when a cover photo is set      |
| `ston-kb-guide-block*` (description, description-wrap, icon-wrap, name, text-wrap)                                              | guide/article block                     |
| `ston-kb-header*` (canvas, contact-us-link, fixed, sticky, subtitle)                                                            | KB header — **don't target from this row alone, see "KB header structure" below** |
| `ston-kb-home`                                                                                                                  | root container of the home page         |
| `ston-kb-logo` / `-logo-link`                                                                                                   | KB logo                                 |
| `ston-powered-by-stonly` / `-custom-text`                                                                                       | "Powered by Stonly" footer badge        |
| `ston-recent-guides-empty-section-text` / `-section-header`                                                                     | "recent guides" section                 |
| `ston-related-link*` (custom-icon, icon, icon-wrap, name, text-wrap), `ston-related-links-grid`                                 | related-link block                      |
| `ston-guide-icon`                                                                                                               | icon next to a guide/article block      |

### KB header structure

A flat "family" listing (e.g. `ston-kb-header*`) hides which class is an actual styleable box vs.
a layout-only wrapper vs. a *separate DOM element* masquerading as a modifier suffix. Confirmed live
(2026-08-19):

```
.ston-kb-header  ← the real background box (a <header>). Target this for header background/border.
  └─ .ston-header-scene  (child here — but reused independently elsewhere, e.g. search results;
       don't assume it's always nested under .ston-kb-header)
       └─ .ston-kb-header-fixed  (layout wrapper only, no own background)
            └─ .ston-kb-header-canvas  (innermost layout wrapper, no own background)
  ├─ .ston-kb-header-subtitle  (sibling text element)
  └─ .ston-kb-header-contact-us-link

.ston-kb-header-sticky  ← NOT a modifier state of .ston-kb-header — a completely separate DOM
  subtree, rendered only once the page is scrolled, with its own duplicate background-color.
  Same scene/fixed/canvas nesting repeats inside it, plus:
  └─ .ston-header-fixed-logo  (logo specifically inside the sticky header's canvas)
```

**Practical takeaway:** to change the header's background color in both states, you need *two*
rules, not one:

```css
.ston-kb-header { background-color: red; }
.ston-kb-header-sticky { background-color: red; }
```

Missing the second rule is a common "my CSS half-works" bug — the header flips back to the old
color the instant the visitor scrolls and the sticky header takes over.

**Don't assume this "separate class" pattern generalizes to every sticky/pinned element.** The
search bar's own sticky-header copy is confirmed to work differently — it's the *same* class with
a mode attribute, not a separate class: `.ston-search-bar[mode="inStickyHeader"]` targets just the
sticky-header copy of the search bar, while plain `.ston-search-bar` (or `.ston-search-bar:not([mode])`
to be explicit) targets the main one. Verify which mechanism applies per element rather than
assuming either pattern by default.

### Favorites section structure

Confirmed live (2026-08-27):

```
.ston-favorite-content-section            ← root <section>; also carries a state modifier class:
  |                                          .loading (spinner only) or .empty (no favorites saved
  |                                          yet), no modifier once populated
  ├─ .ston-favorite-content-section-title  (heading)
  ├─ .ston-favorite-content-loader          (loading state only — stroke-based SVG)
  └─ .ston-favorite-content-list            (populated state only — <ul>)
       └─ <li>                              (no ston-*/plain class of its own — reach via
            |                                 `.ston-favorite-content-list > li`)
            ├─ <a>.ston-favorite-content-title   ← SAME CLASS on two nested elements:
            │    └─ .ston-favorite-content-title   the outer clickable row (<a>) AND, inside it,
            │         ├─ .ston-favorite-content-icon         a smaller icon+text wrapper.
            │         └─ .ston-favorite-content-title-text
            │    └─ .ston-favorite-content-step-title  (sibling, guides only — not shown for
            │                                             standalone articles)
            └─ .ston-favorite-content-remove-button  (opacity:0 by default, shown on row hover/focus)
```

**Practical takeaway:** a rule on `.ston-favorite-content-title` matches *both* the outer link and
the inner icon+text row at once — any box-model property set on it applies twice, nested. To target
only one, use `.ston-favorite-content-title-text` (text only) or scope via a distinguishing child
(e.g. `.ston-favorite-content-title > .ston-favorite-content-icon`).

Icons here are **fill-based** (`path, rect { fill: ... }`); the loader spinner in the same section
is **stroke-based** — the two icon families in one section use opposite SVG coloring, so check
which one you're touching before assuming `fill` (or `color`) works.

Rendering also requires the visitor to be **logged in**, not just the KB's Favorites toggle —
confirmed live: rendering requires both the KB's Favorites toggle being on AND the visitor being
logged in. A logged-out visitor sees nothing (not even the empty state) even with the toggle on.

## Header / navigation

| Class                                                              | Purpose                             |
| ------------------------------------------------------------------ | ----------------------------------- |
| `ston-breadcrumb-arrow` / `-link` / `-list-item`                   | breadcrumb parts                    |
| `ston-breadcrumbs`, `-desktop`, `-list`, `-mobile`                 | breadcrumbs container variants      |
| `ston-hamburger-button`                                            | mobile menu toggle                  |
| `ston-header-menu-canvas`                                          | container of the header menu        |
| `ston-language-switcher`                                           | language-switcher container         |
| `ston-mobile-menu-toggle`, `-bar`, `-bar-1/2/3`                    | hamburger icon parts                |
| `ston-search-button`                                               | search icon/submit button in header |
| `ston-skip-to-main-content`                                        | a11y "skip to content" link         |
| `ston-user-info-email`, `ston-user-menu`, `ston-user-menu-wrapper` | account menu                        |

**The breadcrumb classes above are reused on the guide top bar, not just KB-home/search
navigation** — confirmed 2026-09-07: `ston-breadcrumbs-list` (with `ston-breadcrumb-link`/`-arrow`)
is the same class used above a guide's content, alongside `ston-guide-top-menu-back` (see "Article
/ guide player" table) for the back button and `guide-title` (plain, not `ston-*`) for the title.
Easy to miss since this table's heading reads as KB-shell-only — don't assume a class's section
heading fully scopes where it's actually used.

## Search

| Class                                                                                                                                      | Purpose                         |
| ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------- |
| `ston-applied-*-search-filter` (base, folder, tag), `ston-applied-search-filters-row`                                                      | applied filter chips            |
| `ston-article-search-input*`, `ston-article-search-item-number`                                                                            | in-article TOC search            |
| `ston-custom-metadata-filter-chip` / `-dropdown-option`                                                                                    | custom metadata filter           |
| `ston-extra-result-*-chip` (exact-tag, metadata, tag)                                                                                      | overflow result chips            |
| `ston-filter-chip`, `ston-filter-suggestion(s)`                                                                                            | filter suggestions               |
| `ston-folder-filter-chip` / `-dropdown-no-folders` / `-dropdown-option`                                                                    | folder filter dropdown           |
| `ston-hidden-result-chip` / `-tag-chip`                                                                                                    | chips hidden behind "more"       |
| `ston-kb-search`, `-filter-button` (+ `-filter-button-<mode>` modifier), `-input`, `ston-kb-smart-search` | search page / smart search      |
| `ston-search-filter-<type>` (e.g. `ston-search-filter-folder`) — the filter-**tab** buttons on the full results page, a different element from the `ston-applied-*-search-filter` chips above | search results filter tabs      |
| `ston-more-filters-trigger`, `ston-more-result-chips-trigger`, `ston-more-result-tags-trigger`                                             | "show more" triggers             |
| `ston-result-chip`, `ston-result-tag-chip`                                                                                                 | base result chip                 |
| `ston-results-more` / `ston-search-results-more-container`                                                                                | "Show all results" link — the search **bar's** dropdown, not the full results page, see "Search results page structure" below |
| `ston-search-back-button`, `ston-search-bar`, `ston-search-filter-go-home`, `ston-search-filters`, `ston-search-filters-items-wrap`        | search bar / filters row         |
| `ston-search-in-guide-filter-tabs`                                                                                                         | in-guide search tabs             |
| `ston-search-input`, `-close`, `-submit-button`, `-wrap`                                                                                   | search input field               |
| `ston-search-pagination-numbers` (wrapper), `ston-search-pagination-number` (+ `-active`)                                                  | pagination — **only the numbered page buttons carry a class**; the outer bar, Previous/Next, and the "…" dots don't, see below |
| `ston-search-results-back-link`, `-content`, `-highlight`, `-row` (+`-folder`/`-guide`/`-step`, `-row-highlighted`), `-title`, `-title-wrap`, `-wrap` | result row parts — see "Search results page structure" below |
| `ston-searchbar-with-filters`                                                                                                              | root of search bar with filters  |
| `ston-search-subtitle`, `-wrapper`                                                                                                         | search page subtitle             |
| `ston-tag-filter-chip`, `-dropdown-no-tags`, `-dropdown-option`                                                                            | tag filter dropdown              |
| `ston-tree-search-input*`, `ston-tree-search-item-number`                                                                                  | tree-view sidebar search         |
| `ston-widget-header-filtering`, `ston-widget-search-results-footer`                                                                        | widget-embedded search           |
| `ston-smart-search-text-counter`, `ston-clarification-option`                                                                              | AI smart search                  |

### Search results page structure

Confirmed live (2026-08-31), via a fetch of `/kb/en/search/all/<query>` for several query terms:

```
.ston-search-results-row  ← modifier is literally `ston-search-results-row-${result.type}`,
  |                           type is one of: folder, guide, step (all three confirmed live via
  |                           targeted searches), also explanation, ai per source (unconfirmed live)
  |                         + .ston-search-results-row-highlighted, ADDITIVE on the same row when
  |                           keyboard-focused (paired with role="option"/aria-selected="true" on
  |                           the child link — same ARIA pattern SKILL.md already treats as stable)
  └─ <a role="option">    ← no ston-*/plain class, styled-components hash only:
       |                     [class*="ResultsRowLink"] if you need to target it
       └─ .ston-search-results-content
            ├─ .ston-search-results-title-wrap
            │    ├─ .ston-search-results-title
            │    └─ .ston-page-result-chips-container / -tags-container   (guide rows only)
            │         └─ .ston-chip.ston-result-chip.ston-result-tag-chip
            ├─ .ston-search-results-highlight     (snippet, matched terms wrapped in <b>)
            └─ (breadcrumb trail — no ston-*/plain class either: [class*="ResultsBreadcrumbs"],
                 [class*="Crumb"] per crumb)
  └─ .ston-guide-summary-button   (guide rows only, sibling of the <a>, NOT nested inside it)
```

Between groups of same-type results, a plain (non-`ston-*`) separator label appears —
`className="results-title <type>"` (e.g. `results-title folder`) — real per Classification rule 2,
just not `ston-*`.

**Pagination cannot be seen via any static/raw HTML fetch, regardless of query params — this is a
hydration gotcha, not a fetch failure.** The page's pagination count is only computed client-side
after the initial render, so it's always absent from server-rendered HTML — confirmed by fetching
both the base URL and `?page=2` and finding zero pagination markup in either. If you need to
confirm/style pagination, you must do it against a JS-rendered page (browser DevTools), not a
`curl`/raw fetch. Real classes, from source: only the numbered page buttons carry a class
(`ston-search-pagination-number`, `-active` when current, inside a `ston-search-pagination-numbers`
wrapper) — the outer pagination wrapper, the Previous/Next buttons, and the "…" ellipsis
(skipped-pages) markers all render via styled-components with **no** `ston-*`/plain class, hash
only.

**`ston-results-more`/`ston-search-results-more-container` are a different UI from pagination** —
confirmed live: they're the "Show all results" link inside the search **bar's** inline dropdown
(shown when there are more results than fit), not anything on the full search-results page. Don't
reach for them when styling page pagination.

## Article / guide player (steps, content, footer buttons)

| Class                                                                                                                                                                                                                                                        | Purpose                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| `ston-active-step-content-wrap`                                                                                                                                                                                                                              | active step's content wrapper  |
| `ston-anchor-link`                                                                                                                                                                                                                                           | heading anchor link            |
| `ston-blockquote`, `ston-horizontal-rule`                                                                                                                                                                                                                    | rich-text elements             |
| `ston-checklist-items`                                                                                                                                                                                                                                       | checklist step                 |
| `ston-choice-radio-buttons`, `ston-radio-check`, `ston-radio-wrap`                                                                                                                                                                                           | "next step" radio choice       |
| `ston-copy-code-button` (hover-only copy button) — the code block itself has no class, use `pre[data-type="code-block"]`, see note below | rich-text code block           |
| `ston-compact-illustration`, `-wrap`                                                                                                                                                                                                                         | thumbnail illustration         |
| `ston-completed-steps-button`, `ston-favourites-button`, `ston-feedback-button`, `-title`, `ston-language-button`, `ston-related-content-button`, `ston-table-of-contents-button`, `ston-tts-button`, `ston-comments-button`, `ston-edit-explanation-button` | step-footer action buttons     |
| `ston-content-canvas`, `ston-content-text`, `ston-content-title`                                                                                                                                                                                             | step content                   |
| `ston-content-rating*` (feedback-sent, follow-up-question, negative/positive-button, question, question-container, skip-button, submit-button)                                                                                                               | thumbs up/down rating          |
| `ston-content-summary*` (header, placeholder)                                                                                                                                                                                                                | AI content summary             |
| `ston-enter-fullscreen-button`, `ston-exit-fullscreen-button`, `ston-fullscreen-button`, `ston-fullscreen-image-modal`, `ston-fullscreen-table-modal`                                                                                                        | fullscreen viewers             |
| `ston-file-attachment-*` (icon-wrapper, name, name-wrapper, size, wrapper)                                                                                                                                                                                   | file attachments               |
| `ston-guide-content-canvas`, `-header-canvas`, `-language-row`, `-media-canvas`, `-summary-button`, `-summary-button--widget`, `-top-menu`, `-top-menu-back`                                                                                                 | guide page structure           |
| `ston-illustration`, `-canvas`                                                                                                                                                                                                                               | step illustration              |
| `ston-code-block`                                                                                                                                                                                                                                            | code-illustration/diff media pane — a rendered code snapshot with `added`/`removed`/`modified` styling, **not** the customer's rich-text code block above |
| `ston-image-gallery`, `-video-thumb`                                                                                                                                                                                                                         | image gallery                  |
| `ston-input-file`, `ston-input-wrapper`                                                                                                                                                                                                                      | step input fields              |
| `ston-kb-guide-content`, `ston-kb-guide-header`                                                                                                                                                                                                              | guide/article page             |
| `ston-languages-title`                                                                                                                                                                                                                                       | language list panel heading    |
| `ston-metadata-button`, `-empty-value`, `-label`, `-panel-title`, `-panel-wrap`, `-tag`, `-url-input`, `-value`                                                                                                                                              | metadata panel                 |
| `ston-next-step-button`, `ston-tile-grid`, `ston-selector-types-buttons`                                                                                                                                                                                     | next-step selector             |
| `ston-nps-rating`, `-number`, `-wrap` (+ per-value `ston-nps-rating-<0 through 10>` modifier, one per score) | NPS rating                     |
| `ston-panel-fullscreen-layer`                                                                                                                                                                                                                                | side-panel fullscreen overlay  |
| `ston-previous-step-row`, `ston-previous-steps-title`                                                                                                                                                                                                        | previous steps panel           |
| `ston-progress-bar*` (bare — the fill-track element, nested inside `-wrap` — plus bar, hide-text, line, text, wrap) — see "Known specificity conflicts" for a customer-CSS gotcha on this one | progress bar                   |
| `ston-related-content-*` (external-link, link, link-icon, link-title, link-wrapper, mobile-panel-title, panel, panel-content, panel-content-section\*, placeholder, toggle-button)                                                                           | related content panel          |
| `ston-step-breadcrumbs`, `ston-step-crumb` (**not** `ston-step-breadcrumbs-crumb` — different stem, don't construct it by literal concatenation; confirmed live) | breadcrumbs within a step      |
| `ston-current`                                                                                                                                                                                                                                               | current-language indicator in the guide top bar's language row (`ston-guide-language-row .ston-current`) |
| `ston-step-buttons` (an **`id`**, not a class — see correction below)                                                                                                                                                                                        | "More actions" overflow button only, not the row |
| `ston-step-content-metadata`                                                                                                                                                                                                                                 | metadata block in step content |
| `ston-step-iframe`                                                                                                                                                                                                                                           | embedded iframe in a step      |
| `ston-steps-footer`, `-compact`, `-wrapper`                                                                                                                                                                                                                  | steps footer bar               |
| `ston-tab-switcher-tab`, `-active`                                                                                                                                                                                                                           | tab switcher                   |
| `ston-toc-heading-row`, `-tab-button`, `-tab-panel`, `-toggle-menu-button`                                                                                                                                                                                   | table of contents              |
| `ston-comment-*` (field, logout, post, post-date, sign-in, wrap), `ston-comments-title`                                                                                                                                                                      | comments section               |

**Bonus, non-`ston-*` static class:** the rich-text code block's `<code>` element gets a
`language-${language}` class (e.g. `language-js`) alongside `data-type="code-block"` — confirmed
live (2026-09-07), a Prism/highlight.js-style class, real and plain (rule 2), useful for styling one
language's code blocks differently.

## Contact form

| Class                                                                                   | Purpose      |
| --------------------------------------------------------------------------------------- | ------------ |
| `ston-contact-form-body`, `-canvas`, `-error`, `-header`, `-inputs`, `-modal`, `-title` | contact form |

## Embeddable widget mode

**Two different renderers, not one.** "Widget" content is either a guide or a knowledge base
rendered inside the widget's iframe — they're different React trees with different root classes,
confirmed live (2026-09-08):

| Class                                       | Purpose                                                                                                                                    |
| -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `widget-guide`                               | root class, **guide-in-widget only**, co-occurs with `ston-widget-classic\|-light` on the same element    |
| `ston-widget-classic`, `ston-widget-light`   | resolved widget root variants (from `ston-widget-${light\|classic}`) — **guide-in-widget only**, no equivalent compact-mode class exists for KB-in-widget (compact is a styling-only variant there, not exposed as a class) |
| `ston-widget-header`, `-title`               | widget header — **guide-in-widget only**                                                                                                   |
| `kbWidget-home`                              | root class, **KB-in-widget only**, plain class, no `ston-*` equivalent                                                                     |

**These are all inside the iframe.** Since the content is the customer's own guide/KB, their
styles for this section go in that guide's or KB's own custom CSS field — not anywhere on the
host page.

### Host-page classes (outside the iframe)

A separate set of classes, all plain (not `ston-*`), rendered by the widget embed script on the
**host page**, not inside the iframe — confirmed live (2026-09-08). These only work if the CSS
targeting them is added to whatever page they actually render on: the KB's own custom CSS if the
widget is embedded inside a Stonly KB, or the customer's external page's own stylesheet if the
widget is embedded on an external site. They **cannot** be styled from the guide/KB custom CSS
field that lives inside the iframe (see "Widget/iframe embeds" gotcha in `../SKILL.md`) — the two
are on opposite sides of the iframe boundary.

| Class                        | Purpose                                                                                                                       |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `stn-widget-wrapper`          | widget panel root — co-occurs with `stn-widget-trigger-${widgetRuleId}` (dynamic per-trigger-rule suffix, not a stable "style every widget" hook by itself) |
| `stn-widget-buttons-wrapper`  | wraps the drag/close buttons row                                                                                              |
| `stn-drag-widget-button`      | drag handle button                                                                                                            |
| `stn-close-widget-button`     | close button                                                                                                                  |
| `stn-widget-content`          | content area wrapping the iframe                                                                                             |
| `stn-widget-iframe`           | the `<iframe>` element itself                                                                                                |
| `stn-widget-loader`           | loading spinner shown before the iframe content is ready                                                                     |
| `stn-backdrop`                | modal-mode backdrop overlay                                                                                                  |
| `stn-compact-widget-loader`   | loading placeholder shown before a compact widget's initial open animation                                                   |
| `stn-pill-icon`               | trigger bubble's icon `<img>` — **only present when a custom icon image is uploaded**, absent for the built-in icon set     |

**Gotcha: the trigger button/bubble itself has no stable class.** It's rendered via a hashed
styled-component class — the only hooks on it are `[data-pill-id="..."]` (also a dynamic
per-trigger-rule ID, same caveat as `stn-widget-trigger-*` above) or a generic `[role="button"]`
attribute selector. There is no "style all trigger bubbles at once" class.

**`stonly-widget-id` is not a real class — do not recommend it.** It only appears in an internal
demo page's own scaffolding (a `<code>` element the demo populates itself via `window.STONLY_WID`
in an inline script) — the widget embed script itself never renders this class on any customer's
page.

## Source

Generated by inspecting Stonly's customer-facing frontend for elements carrying the literal
`ston-` prefix (excluding a separate `data-ston-role` attribute pattern used internally for a
different, non-CSS purpose — this exclusion isn't perfectly clean-cut, since that attribute has
occasionally turned up outside its usual place too, but it's not obviously styling-relevant). To
refresh, re-scan the live product for `ston-` classes and diff against this file.
