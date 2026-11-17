# Palquest: front-end review notes

Last updated: 17 November 2026

Findings from reviewing questdev.palestine-studies.org (chronology and home page) on desktop and phone, with suggested fixes for the Drupal theme `bbCustomIPS`.

## Priorities

Fix the bugs and the URL handling before launch; everything else is polish that can follow.

- [ ] Fix the crash on the Themes view (`/en/chronology-thematicevents`)
- [ ] Remove the broken event URL aliases (`/en/overallchronology%3F%26sideid%3D…`) and their Pathauto pattern
- [ ] Put filters, search and the open event in the URL
- [ ] Give each event a real, indexable page with a clean alias
- [ ] Stop rendering all ~2,500 events on page load; load each period on expand
- [ ] Update Drupal from 10.0.9 to a supported release
- [ ] Add the missing font files, or remove the rules that reference them
- [ ] Home page: implement the prototype in `home.html`

## Bugs

| Bug | Where | What happens | Fix |
| --- | --- | --- | --- |
| Themes view crashes | Group by → Themes, or `/en/chronology-thematicevents` loaded directly | Blank page: "The website encountered an unexpected error." The Events view (`/en/chronology-events`) failed once too, then loaded. | Check `/admin/reports/dblog`; likely a PHP exception in the thematic view display. |
| Broken event aliases | `/en/node/6204` (every event) | Redirects to `/en/overallchronology%3F%26sideid%3D6204`. A query string (`?&sideid=`) was put inside a path alias and Drupal encodes it. The canonical tag points there too, so search engines index broken URLs. | Delete those aliases and the Pathauto pattern that creates them; aliases cannot carry query strings. |
| Group by drops state | Group by select | Navigates to the new view without the current query string, so filters and search are lost. | Append `location.search` when navigating. |

## URLs for filters, search and events

Filters already work as URLs on the server; only the address bar is never updated. The exposed forms use `method="get"`, and `/en/chronology-periodsevents?combine=Balfour` returns just the two matching periods. Events also already accept an ID: the event panel's open-in-chronology link points to `/en/chronology-events?nid=12497`.

**1. Shorten the parameter names.** In each Views exposed filter, set "Filter identifier": `field_related_new_subject_target_id` → `subject`, likewise `location`, `actor`, `actor_type`, `q` (for `combine`), `importance`.

```
/en/chronology-periodsevents?q=peel&subject[]=1693&location[]=118
```

**2. Sync the address bar** with a small behavior in `bbCustomIPS`:

```js
(function (Drupal, once) {
  Drupal.behaviors.chronologyUrlState = {
    attach(context) {
      once('chrono-url', 'form[id^="views-exposed-form-chronology"]', context)
        .forEach((form) => {
          form.addEventListener('submit', () => {
            const params = new URLSearchParams(new FormData(form));
            for (const [k, v] of [...params]) if (!v) params.delete(k);
            const url = `${location.pathname}${params.toString() ? '?' + params : ''}`;
            history.pushState({ filters: params.toString() }, '', url);
          });
        });
    },
  };
  // The server renders from GET params, so a reload is the simplest correct back/forward
  window.addEventListener('popstate', () => location.reload());
})(Drupal, once);
```

Better Exposed Filters' auto-submit fires `submit`, so checkbox changes should be caught too; test on the build. The contrib module Views AJAX History does the same job; check Drupal 10 support first.

**3. Single events: two URLs.**

1. A real page per event, canonical and indexable. Pathauto pattern: `/chronology/event/[node:field_date:custom:Y-m-d]-[node:title]` → `/en/chronology/event/1936-05-18-appointment-of-the-peel-commission`. Render the node's full view mode with a "View in timeline" link.
2. Side-panel state on the timeline: event links already carry `data-sideview="6204"`. When `eventSideView` opens the panel, push the event ID to the URL (reuse the existing `nid` parameter); remove it on close.

```js
const url = new URL(location.href);
url.searchParams.set('nid', link.dataset.sideview);
history.pushState({ nid: link.dataset.sideview }, '', url);
```

On load with an event ID: expand the period that holds it, scroll to the row, open the panel. The panel is the `fill_event_container` view filtered by `nid`, so it can also be rendered server-side on first load.

## Chronology page

The page renders the whole chronology up front: about 31,800 DOM nodes and 2,517 images, ~2,500 of them hidden in collapsed periods. Fixing that is the biggest performance win.

**Performance**

- Load each period's events when it is expanded (Views AJAX per period, or a small JSON endpoint).
- Add `loading="lazy"` to every image; only 12 have it now.
- Turn on CSS/JS aggregation in production (the dev page loads 75 stylesheets and 43 scripts).

**Timeline design**

| Issue | Detail | Suggested fix |
| --- | --- | --- |
| Non-linear axis, unmarked | 1500→1900 takes the same width as 1900→1910; the axis starts at 1500 while period I starts in 1516; the last tick is 2023 while period XIII is 2017– | Draw a visible break (//) before 1900, or separate the pre-1900 segment; show start–end years on each bar |
| Titles overlap bars | Period titles run across the green bars; event lines cut through event titles | Put the bar in its own lane or column, never under text |
| Scroll inside scroll | An expanded period scrolls in its own fixed-height box | Let it flow in the page; keep the sticky period header |
| Four title sizes, no key | Event titles appear in ~4 sizes (importance A/B/C) with no legend | Add a legend or a Key/Main/All toggle (already on the Events view) |
| Expand state unclear | "+" stays "+" when open | Switch to − or rotate a chevron; keep `aria-expanded` in sync |
| Year range shifts title on hover | "1947 – 1950" appears on hover and pushes the title right | Show years permanently in their own column |
| Duration bars spill past the period | e.g. PAD Office bar 1961→~2002 inside period V | Clip at the period end with an arrow, or explain in a legend |
| Wasted width | Content stays ~800px wide on large screens | Widen the container to ~1200px; the axis benefits most |

**Mobile:** the intro paragraph fills the first screen, axis labels crowd, and bars sit behind wrapped titles. On phones, drop the Gantt and show a vertical period list with a thin progress strip under each title.

**Smaller ideas:** clicking a year on the axis jumps there; highlight the current scroll position on the axis; auto-expand periods that contain search hits.

## Filters and search panel

Filtering works (AND across groups), but the panel hides its state and the results don't say what was matched.

| Issue | Detail | Suggested fix |
| --- | --- | --- |
| Checked vs unchecked unclear | Unchecked boxes are filled grey squares; checked adds only a tiny tick | Empty outline when unchecked; solid dark green with a white tick when checked |
| Counts unreadable and static | Pale grey on paler pills; USA still shows 178 after ticking Diplomatic | Recompute counts as filters change; disable options that would return 0 |
| Logic not stated | Diplomatic + USA returns events matching both | Say "Events matching all selected groups"; show "Showing 23 of 2,511 events" |
| No date range | The filter people want most on a chronology | Two-handle year slider, or drag across the axis |
| Panel layout | Covers ~80% of the screen with no dim behind; columns fill half; Apply and Reset far apart | Narrow right drawer with collapsible groups, or instant-apply dropdowns (Subject ▾, Location ▾, Actor ▾) |
| Location and Actor look identical | Same region lists in both columns | One-line hint under each heading: "Where it happened" / "Who acted" |
| Generic chip | "2 filters selected" | One removable chip per filter: "Diplomatic ×", "USA ×" |
| Empty periods vanish | Periods with no matches disappear; the axis still starts at 1500 | Grey them out with "0 matches", or fit the axis to the results |
| No per-period count | Filtered periods show outlines only | Add a badge: "4 events" |

**Search box**

- Runs only on Enter, with no button or hint: filter as people type (debounced) or add a search button.
- Results show matching periods still collapsed, with no count or highlighting: open them, highlight the term, show "3 results for 'Balfour'" with a clear (×) button.
- Keep one toolbar across Periods / Themes / Events. Only Events has Display (All/Main/Key) and Sort; search and filters should survive a switch.

**Mobile:** the full-width panel needs a sticky Apply button at the bottom.

## Home page

The working reference is `home.html`; the changes are summarised in the README. Notes for implementing them in Drupal follow.

**Scroll behaviour.** Keep the current layout (banner + Sections / Featured / It happened today); change only how it scrolls. Today the Featured column is a 900px box scrolling inside a page that also scrolls. In the new behaviour only one thing scrolls at a time:

1. The page scrolls: the banner stays fixed and the columns slide up over it.
2. The columns reach the header and stop. The panel is exactly one screen tall, so the page has nothing more to scroll; the header turns white and shows the small logo.
3. Each column now scrolls on its own. Column titles stay pinned.
4. Scrolling up at the top of a column hands the scroll back to the page (native scroll chaining): the columns slide down and the banner returns.

Columns stay locked (`overflow: hidden`) until the panel docks; otherwise a column under the pointer grabs the wheel while the panel is still rising.

```css
:root { --hdr: 60px; --hero-h: 74svh; }
.hero  { position: fixed; inset: 0 0 auto; height: 100svh; z-index: 0; }
.panel {
  position: relative; z-index: 10; margin-top: var(--hero-h);
  height: calc(100dvh - var(--hdr));          /* docks exactly under the header */
  display: grid; grid-template-columns: repeat(3, minmax(0, 1fr));
}
.col { min-height: 0; overflow: hidden; }
body.is-docked .col { overflow-y: auto; }
.col__title { position: sticky; top: 0; z-index: 3; background: #fff; }
.site-header { position: fixed; inset: 0 0 auto; height: var(--hdr); z-index: 30; }
body.is-docked .site-header { background: #fff; box-shadow: 0 1px 0 #dfe5e1; }

@media (max-width: 960px) {             /* phones: one column at a time, in tabs */
  .panel { height: auto; display: block; }
  .col, body.is-docked .col { overflow: visible; }
}
```

```js
(function (Drupal, once) {
  Drupal.behaviors.homeDock = {
    attach(context) {
      once('home-dock', '.panel', context).forEach((panel) => {
        const hdr = parseFloat(getComputedStyle(document.documentElement).getPropertyValue('--hdr'));
        const update = () => document.body.classList.toggle('is-docked',
          panel.getBoundingClientRect().top <= hdr + 1);
        window.addEventListener('scroll', update, { passive: true });
        window.addEventListener('resize', update);
        update();
      });
    },
  };
})(Drupal, once);
```

**Details**

- Scrollable columns get `tabindex="0"` and `aria-labelledby` pointing at their title, so keyboard users can scroll them.
- Footer: once the panel docks there is no room for it. Move its links into the menu, or end each column with an "All … →" link (as in the prototype).
- RTL: under `dir="rtl"` the grid flips on its own; the panels slide in from the left.
- Header and banner content use the same container and side margins as the columns (`max-width: 1320px` plus a 12px inset).

**Other fixes**

| Area | Now | Change |
| --- | --- | --- |
| Page title | Tab reads "\| Interactive Encyclopedia…" (empty title field) | Set a real title for the front page |
| Main heading | No `<h1>` | Make the banner logo the `<h1>` (an image with the full title as alt text) |
| Images | Mixed ratios; tall cards in Featured; originals up to 18 MB | One ratio (3:2) in Featured with `object-fit: cover`; serve resized images; add alt text (3 of 13 images have none) |
| Card labels | "HIGHLIGHT" / "BIOGRAPHY" render in Times (missing font) | See Fonts |
| Meta | No meta description or share image | Add a meta description and an `og:image` |

## Fonts

The theme loads two fonts but references four more that have no font file, so those rules fall back to Times, Arial or Helvetica. All rules are in `themes/custom/bbCustomIPS/css/style.css`.

| Font | File loaded? | Used by | Falls back to |
| --- | --- | --- | --- |
| Lyon Text | Yes: `fonts/LyonArabicText-Regular-Web.woff2` | Serif titles | — |
| Suisse Intl | Yes: `fonts/SuisseIntl-Book.woff2` | Body and UI text | — |
| PP Neue Montreal | No | `.path-frontpage .featuredblock .landingcard-item .carddata .card-type` ("HIGHLIGHT" / "BIOGRAPHY" labels on home cards) | Times |
| Fedra Serif Arabic Medium | No | `#sidemenu .menu--main h2, #sidemenu .menu--main .block-title` (side menu headings) | Times |
| Fedra Sans Arabic Bold | No | `.platformHeaderdiv .platformtitlediv`; `.nav-tabs.papers_tab li` | Arial |
| Fedra Sans Arabic Book | No | `#sidemenu .menu--main ul li .dropdown-menu li a` (side menu sub-links) | Arial |

- [ ] Decide per rule: add the licensed web font files (Fedra: Typotheque; PP Neue Montreal: Pangram Pangram) with `@font-face`, or switch the rule to Suisse Intl / Lyon Text.
- [ ] Only one weight of each is loaded (Lyon Regular, Suisse Book). Anything set bold or medium is synthesized by the browser. Add Suisse Medium (and Lyon Italic if used), or set `font-synthesis: none`.

Both loaded fonts are commercial and licensed for the site's domain; don't publish the font files elsewhere.

## Accessibility

The basics exist (skip link, `<main>`), but headings, alt text and some controls need work.

- [ ] Chronology page has no `<h1>`: mark up "Chronologies" as the `<h1>`
- [ ] Home page has no `<h1>` (see Home page)
- [ ] Chronology: ~2,500 of 2,517 images have no `alt`; use the captions already on the page
- [ ] Home: 3 of 13 images have no `alt`
- [ ] 3 controls on the chronology page have no accessible name (likely the search icon and the mobile menu button)
- [ ] Expand/collapse buttons: keep `aria-expanded` in sync with the open state
- [ ] Check contrast of the grey event dates and the filter count pills against WCAG AA (4.5:1)
- [ ] Scrollable regions (home columns) need `tabindex="0"` and a label
- [ ] Visible focus outline on every link and control; respect `prefers-reduced-motion`

## Stack and housekeeping

The front end is server-rendered Drupal with jQuery and no JS framework. Keep it that way for a multilingual (EN/AR, RTL), search-dependent encyclopedia; the problems come from patch scripts and rendering everything at once, not from the stack.

| Item | Now | Action |
| --- | --- | --- |
| Drupal core | 10.0.9 (the 10.0 branch no longer gets security updates) | Update to a supported 10.x or 11 release |
| Theme | `bbCustomIPS`, subtheme of Bootstrap Barrio; Bootstrap 5.2 from jsDelivr | — |
| jQuery | Loaded twice: 3.5.1 slim from code.jquery.com + core 3.6.3 | Remove the CDN copy |
| Views | `chronology_periods` (`page_3`) + `fill_event_container` (`block_1`, side panel); Better Exposed Filters; custom `ips_facet_counts`, `xmlk_suggest` | — |
| Custom scripts | ~15 files such as `fix-advancedfilters-finals4.js`, `fix-importance-field41.js`, `timeline_refactorednew.js`, `myLeafletArcGISLayerfinal54.js` | Merge into one versioned `chronology` library attached only on chronology routes; cache-bust with `version:` in `.libraries.yml`, not by renaming files |
| Unused libraries | DataTables, Select2 and the Leaflet/ArcGIS script load on the chronology page | Attach them only where used |
| Aggregation | 75 stylesheets, 43 scripts on dev | Enable CSS/JS aggregation in production |
| Cloudflare challenge | "Verify you are human" on questdev | Fine for dev; make sure it doesn't carry over to production, where it would block crawlers and link previews |

## Prototype files

| Path | What it is |
| --- | --- |
| `home.html` | Home page prototype (desktop and mobile) |
| `assets/web/hero.jpg`, `hero-2.jpg`, `hero-3.jpg` (+ `-1200` versions for phones) | Banner photos with the tint from `hero-homepage.svg` (75% #06281E + multiply #C7D7CD) |
| `assets/web/*.jpg` | Featured and "It happened today" photos resized to max 1200px |
| `assets/web/logo-2l-*.svg`, `logo-4l-*.svg`, `logo-mobile-dark.png` | Logos: `light` (white) for the banner, `dark` for the white header; compact logo for phones |
| `assets/web/logo-ips.svg`, `ips_sidemenu_logo.png`, `close.svg` | IPS logos and the menu close icon |
| `assets/fonts/` | Lyon Text and Suisse Intl from the theme (licensed; not for publication) |
| `assets/` (originals) | Source files supplied by the designer |

The prototype uses text from the dev site on 17 November 2026; links don't navigate.
