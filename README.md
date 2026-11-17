# Palquest front-end review

Design and interaction proposals for the Interactive Encyclopedia of the Palestine Question. Each page is a static prototype: open the HTML file in a browser. Images and fonts are in `assets/`.

## Home (desktop and mobile)

Prototype: [home.html](home.html)

**Scrolling**
- The page scrolls more naturally. The intro banner stays in place and the three columns slide up over it.
- Once the columns reach the top, the navigation bar appears and each column scrolls on its own.
- Column titles stay pinned while scrolling, with the labels "Editors' selection" and "17 November".
- A white gradient at the bottom of each column shows there is more to scroll.
- On mobile, the three columns become tabs.

**Banner and navigation**
- The IPS logo is in the bottom-left of the banner, labelled "A project by".
- The logo and text in the banner and navigation bar now share the same side margins as the columns below, so everything lines up.
- The banner is a series of archival photos that fade slowly into each other every 5 seconds, instead of a single image. This gives the site a stronger visual identity and shows more of the archive.
- The language switch is now a dropdown (English, Arabic).
- On mobile, the navigation bar uses the compact logo: English on the English site, Arabic on the Arabic site. The logo files can be provided again if needed.

**Sections**
- Search is now listed with the sections, after Documents, and opens a search box in place. Search is one of the main ways into the encyclopedia, so it should be on the home page.

**Event panel**
- The panel closes when you click anywhere outside it, as well as with Close or the Escape key.
- An "Open event in chronologies" button replaces the small icon next to the title. A labelled button makes it clearer what the link does.

**Hover and click**
- The whole row or card is clickable, not only the text. This also makes items easier to tap on mobile.
- Hovering a section or event highlights the row in light green and shows an arrow.
- Hovering a featured item shows "Read highlight →" or "Read biography →".
- Images in Featured and It happened today zoom in slightly on hover.

## Developer notes

Bugs, URL handling, chronology and filter issues, fonts, accessibility and Drupal housekeeping, with suggested fixes: [docs/developer-notes.md](docs/developer-notes.md)
