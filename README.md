# Wolfenstein 3D Map Viewer

A simple single-page viewer for Wolfenstein 3D level maps. Pick any of the 60 levels (6 episodes × 10 floors) and see its map, a description, key stats, strategy notes and secrets, with quick links to the wiki article and a Steam guide.

<!-- Add your live link here once GitHub Pages is switched on -->

## Features

- **All six episodes:** 10 floors each, including the Boss (Floor 9) and Secret (Floor 10) levels
- **Easy navigation:** Back/Forward buttons, a dropdown, and a table of quick links at the bottom of the page
- **Zoom:** click the map for a sharp, pixelated 200% view
- **Live wiki info:** for each level the page pulls the description, stats table, Strategy and Secrets sections from the Wolfenstein Wiki, and remembers them so each level is only fetched once
- **Fast switching:** the next and previous maps are quietly preloaded
- **Steam guide link:** opens the guide at the section for the current level
- **Strategy warning** for Episode 4, Floor 2, where the order you push the secret walls matters (adapted from a Steam community post)

## Keyboard shortcuts

| Key | Action |
| --- | --- |
| `←` / `→` | Previous / next level |
| `Space` or `Enter` | Open the zoomed map |
| `Space`, `Enter` or `Esc` | Close the zoomed map |

## Running it

There is nothing to build or install. Open `wolf3d_map_viewer.html` in a browser. It needs an internet connection, because the maps and level info are loaded from the wiki.

To host it on GitHub Pages, rename the file to `index.html`, then go to **Settings → Pages**, choose the `main` branch and save.

## How it works

- **Maps:** the images are loaded directly from the wiki's image server. Each file's folder is the first character, then the first two characters, of the MD5 hash of its file name (for example `WOLF3D_E2M01.PNG` is in `a/ad/`). These folders are stored in the `mapPaths` list at the top of the script.
- **Level info:** when the level changes, the page asks the wiki's MediaWiki API for that level's page and picks out the intro paragraph, the stats table, and the Strategy and Secrets sections. If the normal request is blocked by the browser, it falls back to a script-tag request.
- **Safety:** wiki content is cleaned before it is shown. Scripts, images and attributes are removed, and only links to other wiki pages are kept.

## Configuration

All settings are at the top of the `<script>` block.

| Setting | What it does |
| --- | --- |
| `mapPaths` | Image folder for each level |
| `steamSectionIds` | Steam guide section ID for each level |
| `STEAM_HASH_PREFIX` | What goes between the guide address and the section ID (currently `#`) |

<details>
<summary>Getting the Steam section IDs</summary>

Open the Steam guide, press F12, go to the Console tab and run this. Then paste the output into `steamSectionIds`.

```js
var options = document.querySelectorAll('.rightbox_list_option');
var lines = [];
for (var i = 0; i < options.length; i++) {
    var text = options[i].textContent.trim();
    var match = text.match(/Episode (\d+) - Floor (\d+)/);
    if (match) {
        var floor = match[2];
        if (floor.length < 2) { floor = '0' + floor; }
        var id = options[i].id.replace('guideSectionSelection_', '');
        lines.push('"E' + match[1] + 'M' + floor + '": "' + id + '",');
    }
}
console.log(lines.join('\n'));
```

</details>

<details>
<summary>Optional: make Steam jump straight to the section</summary>

Steam does not always scroll to a section from the address alone. With a userscript manager such as Tampermonkey or Violentmonkey, this script makes the guide page jump to the right section when opened from this viewer:

```js
// ==UserScript==
// @name         Steam guide - jump to section from #id
// @match        https://steamcommunity.com/sharedfiles/filedetails/*
// @grant        none
// ==/UserScript==

function jumpToSection() {
    let sectionId;

    sectionId = window.location.hash.replace("#", "");
    if (/^\d+$/.test(sectionId)) {
        setTimeout(function () {
            SelectGuideSection(sectionId, "changeHash");
        }, 500);
    }
}

if (document.readyState === "complete") {
    jumpToSection();
} else {
    window.addEventListener("load", jumpToSection);
}
```

</details>

## Credits

- Maps and level information: [Wolfenstein Wiki](https://wolfenstein.fandom.com)
- Secret map guide: [Maps and secrets (episode 1..6)](https://steamcommunity.com/sharedfiles/filedetails/?id=1325446230) on Steam
- Episode 4, Floor 2 wall-pushing order: from a post by a member of the Steam community

Wolfenstein 3D is © id Software / Bethesda. This is an unofficial fan project and is not affiliated with or endorsed by them. The wiki's text and images are community-contributed, so check the wiki's licence before reusing them elsewhere.
