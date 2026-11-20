# Chart Busters: Music Style Explorer

An interactive prototype for exploring music taste by **audio attributes** (mood, energy, danceability, acousticness, tempo) rather than by genre or artist.

Two more pages: **Artist profiles** (`artist-profiles.html`), and **Distributions & keys** (`key-wheel.html`): ridgeline charts of how song attributes are spread out, followed by *Études in twelve keys*, a circle of fifths showing which keys popular songs use.

You search for a song you like, adjust its attributes, get song recommendations, and build a **spider chart** of your style from the songs you choose.

> Status: course prototype (UW, Fall 2026). Single HTML page, no build step, no dependencies.

---

## Live demo

**https://trtr04.github.io/chartbuster/** (hosted on GitHub Pages from the `main` branch)

---

## Quick start

The page loads its data from a separate file, so serve the folder rather than double-clicking the HTML:

```bash
git clone https://github.com/trtr04/chartbuster.git
cd chartbuster
python3 -m http.server 8765
```

Then open **http://localhost:8765/style-explorer.html**.

Any static server works (`npx serve`, VS Code Live Server, and so on).

---

## How to use it

The page reads top to bottom: **search → explore (sliders + song map) → recommendations → your style profile**. At first only the search bar shows; the rest appears once you pick a song (or click *"or start exploring without a song"*).

| Step | What you do | Success | Failure |
|---|---|---|---|
| **① Search for a song** | Type in the search box (it autocompletes), e.g. *"All I Want for Christmas"*. | Sliders animate to that song's values and the song is ringed on the song map. | A **"Song not found"** screen suggests similar titles, or lets you skip and use the sliders directly. |
| **② Tune the attributes** | Drag the vertical sliders (or use the arrow keys / PageUp / PageDown). Uncheck an attribute to ignore it. Adjust **match tolerance** under *Advanced*. | **Our recommendations** lists ranked matches (1, 2, 3, …) and the matches light up on the song map. | **"No songs match"**: the previous results stay in place, greyed out, with buttons to widen the tolerance or jump to the closest song. |
| **③ Build your spider chart** | Tick **Add to chart** on any recommendation, or press ▶ to play it (played songs are added automatically). | The spider chart shows the average of your chosen songs, plus a **genre breakdown** (e.g. *"67% are Pop, 17% Rock and 17% Rap"*) and **More for you, across genres**: one recommendation per genre matched to your profile, with genres you haven't tried marked *new for you*. | Before any song is added, an empty state explains what to do. |

Other interactions:

- **Song map** (scatter plot of all 900 songs): hover a dot for details, click a dot to use that song as your baseline, and switch either axis with the X / Y dropdowns.
- **▶ Play** opens Spotify's embedded player in a dock at the bottom-right (a preview, or the full song if you are logged in to Spotify).
- **Top bar** (on every page, stays pinned while you scroll): switch between *Style Explorer*, *Artist profiles* and *Distributions & keys*. Its **Dark / Light** button switches the theme on all pages and remembers your choice; by default the site follows your system setting.
- **Reset to song** returns every slider to the searched song's values.
- Hover a song in the recommendations or profile list to overlay it on the spider chart.
- A "Show as table" option under the spider chart lists the same values in a table.

### Color legend

Colors mean the same thing everywhere (sliders, song map, spider chart):

| Color | Meaning |
|---|---|
| 🟦 Blue | The **searched song's** value |
| 🟧 Orange | **Your change** / your target range |
| 🟩 Aqua | **Recommendations** and **your chart** (average of chosen songs) |

On a slider, a blue square marks the song's original value. Once you drag away from it, your handle turns orange and an orange line shows the difference.

### Artist profiles (`artist-profiles.html`)

One page per artist for 1,035 artists with 20+ tracks: a sound fingerprint (each attribute vs. the average artist, with percentile), a mood map of their top tracks, hits vs. deep cuts, top tracks and genres, and "Sounds like" artists you can click through. Search or press *Random artist*.

- Open it from the top bar, or click an **underlined artist name** in the explorer (search pill, recommendations, *More for you*). Names are only linked when that artist has a profile.
- Deep link: `artist-profiles.html?artist=Katy%20Perry`. An unknown name shows a note and falls back to The Weeknd.
- The page is self-contained (its data is inlined, about 1.5 MB). `data/artist_names.js` lists which artists have a profile; if the page's data changes, rebuild it with `python3 data/build_artist_names.py artist-profiles.html data/artist_names.js`.

### Distributions & keys (`key-wheel.html`)

The page has two parts, top to bottom.

**1. Song attribute ridgelines.** One smooth curve (density) per row, every attribute on a 0–1 scale. The color inside each curve is the average popularity of songs at that value; the tick marks the median.

- **Dataset**: the explorer's 900 songs, or the 114k-track Spotify dataset (binned).
- **View**: all attributes for one genre, one attribute across genres, or one attribute across year ranges (900 songs only, since the 114k set has no years).
- Hover for counts, median, middle half, and the share of songs below the cursor. *How to read this chart* explains the scaling.
- Data: `data/ridgelines.js` (about 2 MB). Uses d3 from a CDN.

**2. Études in twelve keys** (below the ridgelines):

A circle of fifths built from about 90k Spotify tracks. Open the page from the top bar; *Skip to Études in twelve keys ↓* jumps past the ridgelines.

- **Outer ring = major keys, inner ring = their relative minors.** Bar length is the share of songs in that key; both rings use the same scale.
- **Songs** dropdown: filter by popularity (default *Hits: popularity 60+*). The dashed outline shows every song in the same genre family, so you can see which keys hits favor.
- **Genre chips / Movements**: nine genre families (Pop, Hip-hop & R&B, Rock, Metal & punk, Electronic, Latin, Country & folk, Jazz & blues, Classical & calm), each also drawn as a small wheel.
- **Click a key** for its key signature, share, average mood / energy / BPM, the most popular songs in it (links to Spotify), and a ▶ button that plays the chord.
- Three notes under the wheel sum up the current view: most common key, major vs. minor split, and whether minor songs score sadder on mood.

---

## Data

**Source:** [Spotify Most Popular Songs Dataset](https://www.kaggle.com/datasets/rishabhpancholi1302/spotify-most-popular-songs-dataset) on Kaggle (`songs.csv`, 900 tracks).

- Mostly releases from **2015–2020** (newest: January 2020); genres: Pop, Rap, Latin, R&B, Rock, EDM.
- The audio features in the source are **min-max scaled to 0–1 across the dataset**. The app multiplies them by 100, so every value is a **relative** score within these 900 songs:
  - **Tempo** is "slower ↔ faster", not beats per minute.
  - **Popularity** 50 means "middle of this set", not Spotify's absolute popularity score.
- **Mood** is Spotify's `valence` (sad → happy).

### Regenerating the data file

`data/spotify_songs.js` is a trimmed copy of the CSV, about 100 KB instead of 20 MB; it drops lyrics, tags and vectors. To rebuild it after downloading `songs.csv` from Kaggle:

```bash
python3 data/build_data.py path/to/songs.csv data/spotify_songs.js
```

The raw CSV is not committed (see `.gitignore`).

### Key wheel data

**Source:** [Spotify Tracks Dataset](https://www.kaggle.com/datasets/maharshipandya/-spotify-tracks-dataset) on Kaggle (`dataset.csv`, 114,000 rows, 114 genres, 89,741 unique tracks). It has Spotify's estimated `key` and `mode` (major/minor), real BPM `tempo`, and a 0–100 `popularity` score, but no release dates.

`data/key_wheel.js` (about 150 KB) holds counts per key, mode, popularity band and genre family, plus the top five songs per key. Rebuild it with:

```bash
python3 data/build_key_data.py path/to/dataset.csv data/key_wheel.js
```

Key and mode are automatic estimates; Spotify often confuses a major key with its relative minor, so the major/minor split is approximate.

---

## Project structure

```
.
├── index.html                 # Redirects to style-explorer.html (GitHub Pages entry)
├── style-explorer.html        # The whole app: HTML, CSS and JS
├── artist-profiles.html       # Artist profiles page (data inlined)
├── key-wheel.html             # Distributions & keys: ridgelines + circle of fifths
├── data/
│   ├── spotify_songs.js       # Trimmed dataset (window.SONG_DATA)
│   ├── build_data.py          # CSV → JS converter
│   ├── artist_names.js        # Artists that have a profile (for explorer links)
│   ├── build_artist_names.py  # artist-profiles.html → artist_names.js
│   ├── ridgelines.js          # Ridgeline data (900 songs CSV + binned 114k tracks)
│   ├── key_wheel.js           # Key/mode aggregates (window.KEY_DATA)
│   └── build_key_data.py      # dataset.csv → key_wheel.js
├── Concept_proposal_Nov5.docx # Original concept proposal
└── .claude/launch.json        # Local dev-server config (optional)
```

### How matching works

For every attribute that is checked, a song matches if it is within **± tolerance** of your slider value. Matches are ranked by root-mean-square distance across the checked attributes, with popularity as the tie-breaker, and the top 8 are listed.

---

## Not yet built / known limitations

- **Time-series view of popular songs** (part 1 of the original whiteboard plan) is not implemented yet.
- Recommendations come only from the 900 songs in the dataset, which skews toward 2019.
- Spotify's embedded player needs an internet connection; album art loads from Spotify's image CDN.
- Taste profiles are not saved; reloading the page clears them.
