# Palquest front-end review

Design and interaction proposals for the Interactive Encyclopedia of the Palestine Question. Each page is a static prototype: open the HTML file in a browser. Images and fonts are in `assets/`.

## Home (desktop and mobile)

Prototype: [home.html](home.html)

**Scrolling**
- The page scrolls more naturally. The intro banner stays in place and the three columns slide up over it.
- Once the columns reach the top, the navigation bar appears and each column scrolls on its own.
- Column titles stay pinned while scrolling, with the labels "Editors' selection" and "17 November".
- A white gradient at the bottom of each column shows there is more to scroll.

**Banner and navigation**
- As requested the IPS logo is in the bottom-left of the banner, labelled "A project by".
- The logo and text in the banner and navigation bar now share the same side margins as the columns below, so everything lines up.
- The banner is a series of archival photos that fade slowly into each other every 5 seconds, instead of a single image. This gives the site a stronger visual identity and shows more of the archive.
- The language switch is now a dropdown (English, Arabic). The current language has a checkmark, and Arabic is written in Arabic script so Arabic speakers can find it.

**Sections**
- Search is now listed with the sections, after Documents, and opens a search box in place. Search is one of the main ways into the encyclopedia, so it should be on the home page.

**Featured**
- All featured images use the same 3:2 crop, so the column looks even instead of mixing tall and wide images.
- The "Highlight" and "Biography" labels use the site's font (Suisse). On the live site they show in Times because a font file is missing.

**Event panel**
- The panel closes when you click anywhere outside it, as well as with Close or the Escape key.
- An "Open event in chronologies" button replaces the small icon next to the title. A labelled button makes it clearer what the link does.

**Hover and click**
- The whole row or card is clickable, not only the text. This also makes items easier to tap on mobile.
- Hovering a section or event highlights the row in light green and shows an arrow.
- Hovering a featured item shows "Read highlight →" or "Read biography →".
- Images in Featured and It happened today zoom in slightly on hover.

**Mobile**
- The three columns become tabs (Sections, Featured, Today), so each list is one tap away instead of a long scroll.
- The tab bar stays pinned under the navigation bar, and switching tabs jumps back to the top of the column.
- The navigation bar uses the compact logo that was developed in the identity design phase: English on the English site, Arabic on the Arabic site. The logo files can be provided again if needed.

**Performance**
- Try to reduce the size of images if possible to reduce the loading time. 

**Accessibility notes (general)**
- Everything works with the keyboard: Escape closes the panel and menus, and focus returns to where you were.
- Every image has a text description, and the banner logo is the page's main heading.
- Animations are reduced, and the banner photos stop changing, for visitors who turn motion down in their system settings.
