# iwcommunitymap
IW Community Map
IW Clickable US Map (with OCONUS call-outs)
An embeddable, clickable SVG map of the United States sized for web pages and event hubs. Includes optional “OCONUS” call-outs (e.g., mainland France, Puerto Rico) positioned near the Southeast coast of the US.
Labels use a lighter olive. Built by Nick Sawall utilizing CHatGPT 5. 


Background uses a light/white/olive canvas (configurable).


Links open chapter pages (same-tab by default).

Star for DC

Europe (France) call-out sits off the Florida/GA/SC coast and is a bit larger for readability.



Quick start
Open locally


Double-click index.html, or run a tiny server (optional):


Python: python -m http.server 8000


Node: npx http-server .


Visit http://localhost:8000


Build/Deploy


Static site—just host these files (e.g., GitHub Pages, Netlify, S3).


If using GitHub Pages, push to main and enable Pages in repo settings.



Project structure
.
├─ index.html
├─ styles.css
├─ /src
│  ├─ map.js            # boots the map, wiring, click handlers
│  ├─ labels.js         # label placement rules
│  ├─ oconus.js         # OCONUS call-out geometry placement + toggles
│  └─ utils.js
├─ /config
│  └─ chapters.json     # chapter list + URLs (+ on/off switch per OCONUS)
├─ /data
│  ├─ us-states.svg     # base US map (AK/HI inset)
│  └─ oconus
│     ├─ france-mainland.svg
│     ├─ puerto-rico.svg
│     ├─ guam.svg               # currently disabled
│     └─ st-thomas.svg          # currently disabled
└─ /assets
   └─ icons.svg
If your file names differ, keep the paths consistent with the examples below.

Configuration cheatsheet
Map size (outer SVG viewport): in index.html, look for:


<svg id="map" viewBox="0 0 1200 750" preserveAspectRatio="xMidYMid meet">
Increase both numbers to make more room (e.g., 1400 850) if Europe/PR feel cramped.


Background + palette: in styles.css:


body { background: #f7f7f2; }        /* light/white/olive */
.label { fill: #556b2f; }            /* lighter olive labels */

How chapters are wired
1) 
config/chapters.json
Central source of truth for chapter names and URLs (including OCONUS). Example:
{
  "states": [
    { "id": "TN", "name": "Tennessee", "url": "https://example.com/tn" },
    { "id": "GA", "name": "Georgia",   "url": "https://example.com/ga" }
  ],
  "oconus": [
    { "id": "france-mainland", "name": "France", "url": "https://example.com/fr", "enabled": true },
    { "id": "puerto-rico",     "name": "Puerto Rico", "url": "https://example.com/pr", "enabled": true },

    { "id": "guam",            "name": "Guam", "url": "https://example.com/gu", "enabled": false },
    { "id": "st-thomas",       "name": "St. Thomas", "url": "https://example.com/stt", "enabled": false }
  ]
}
2) 
/data/oconus/*.svg
Tiny, self-contained SVGs for each OCONUS outline. If you introduce a new region, add its SVG here.
3) 
src/oconus.js
Positions and scales each OCONUS outline in the US canvas. Example entries:
export const OCONUS_PLACEMENTS = {
  "france-mainland": {
    svgPath: "data/oconus/france-mainland.svg",
    // placed to the right of FL/GA/SC
    x: 980,  // move right (+) / left (-)
    y: 420,  // move down (+) / up (-)
    scale: 1.2,    // make bigger/smaller
    labelDx: 12,   // nudge label horizontally
    labelDy: 4     // nudge label vertically
  },
  "puerto-rico": {
    svgPath: "data/oconus/puerto-rico.svg",
    x: 930,
    y: 530,
    scale: 1.35,
    labelDx: 10,
    labelDy: 2
  },

  // currently disabled, but kept for future use
  "guam": {
    svgPath: "data/oconus/guam.svg",
    x: 1060, y: 590, scale: 1.8, labelDx: 8, labelDy: 2
  },
  "st-thomas": {
    svgPath: "data/oconus/st-thomas.svg",
    x: 970, y: 560, scale: 2.0, labelDx: 6, labelDy: 2
  }
};
If something overlaps (e.g., Europe touching Tennessee), bump the x/y to reposition or increase the outer viewBox in index.html.
4) 
src/labels.js
Auto-generates labels for states and OCONUS from chapters.json. Provides default offsets; you can override with per-item labelDx/labelDy in OCONUS_PLACEMENTS.

Add or remove OCONUS chapters
To 
add
 an OCONUS chapter
Add chapter record (enabled) in config/chapters.json:


{ "id": "england", "name": "England", "url": "https://example.com/england", "enabled": true }




Add geometry: create /data/oconus/england.svg.


Keep it simple: single <path> inside an <svg> with viewBox set (e.g., 0 0 100 60).


Position it in src/oconus.js:


"england": {
  svgPath: "data/oconus/england.svg",
  x: 1005, y: 400,  // start near the SE coast cluster
  scale: 1.2,
  labelDx: 10, labelDy: 2
}




(Optional) Label nudge: tweak labelDx/labelDy if the label kisses another element.


Test:


Reload the page.


Click should open the chapter URL set in chapters.json.


If it overlaps, increase viewBox (e.g., 1200×750 → 1400×850) or adjust x/y.


Tip: Search for OCONUS_PLACEMENTS in src/oconus.js to jump to the exact block to edit.

To 
disable/remove
 an OCONUS chapter
Soft remove (recommended):
Set "enabled": false for that item in config/chapters.json:


{ "id": "guam", "name": "Guam", "url": "...", "enabled": false }




Leave the SVG and placement in place—keeps history and makes re-enabling a one-line change.


Hard remove:
Delete the object from config/chapters.json.


Remove the entry from OCONUS_PLACEMENTS in src/oconus.js.


(Optional) Delete /data/oconus/<id>.svg.



Common adjustments
Europe overlaps the Southeast / Tennessee?


Nudge France (france-mainland) in src/oconus.js: increase x to move it farther right, increase y to move it down.


If still tight, bump viewBox in index.html.


Puerto Rico sits over Florida?


In src/oconus.js, increase PR’s x by ~+20–40 and/or y by ~+10–20.


Example: from { x: 930, y: 530 } → { x: 960, y: 545 }.


Make the main window bigger


index.html → <svg id="map" viewBox="0 0 1400 850"> (or larger).


If labels look small, you can scale font via .label { font-size: 12px; } in styles.css.


Change label color to lighter olive


styles.css:


.label { fill: #556b2f; }  /* adjust as desired */




Remove helper notes (e.g., “links go to…”)


In index.html, delete the helper <p> or comment block near the map container.



Accessibility & UX
States and OCONUS call-outs are focusable; pressing Enter triggers the link.


Add accessible names via aria-label if you introduce new regions.


Keep sufficient contrast for labels against the light background.



Troubleshooting
Nothing happens when clicking an OCONUS item


Confirm enabled: true in chapters.json.


Confirm id matches between chapters.json and OCONUS_PLACEMENTS.


Check the svgPath under data/oconus/.


Label overlaps a path


Increase labelDx/labelDy for that entry.


Consider a slight scale increase and reposition.


Mobile overflow/clipping


Ensure the parent container allows overflow: visible.


Maintain preserveAspectRatio="xMidYMid meet".



Current OCONUS status
Enabled: mainland France, Puerto Rico


Disabled (kept for future): Guam, St. Thomas



Maintenance notes
Keep geometry simple to minimize file size.


Prefer soft removes by toggling "enabled" in chapters.json so you can restore quickly.


Commit meaningful messages like feat(oconus): add england call-out at SE cluster.



Need exact “which line to change?” guidance later?
Search within the repo:
index.html → viewBox="0 0


src/oconus.js → OCONUS_PLACEMENTS


config/chapters.json → "oconus": [


I kept all edits centralized so you won’t have to redo the whole code when adjusting positions. If you want, tell me your current viewBox and the x/y you see for France/PR, and I’ll give you precise new values.

