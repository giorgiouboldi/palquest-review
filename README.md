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
- The banner photos change every 5 seconds with a slow fade.
- The language switch is now a dropdown (English, Arabic).
- On mobile, the navigation bar uses the compact logo: English on the English site, Arabic on the Arabic site. The logo files can be provided again if needed.

**Hover and click**
- The whole row or card is clickable, not only the text. This also makes items easier to tap on mobile.
- Hovering a section or event highlights the row in light green and shows an arrow.
- Hovering a featured item shows "Read highlight →" or "Read biography →".
