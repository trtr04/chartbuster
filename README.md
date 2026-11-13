# Chart Busters: Music Style Explorer

An interactive prototype for exploring music taste by **audio attributes** (mood, energy, danceability, acousticness, tempo) rather than by genre or artist.

You search for a song you like, adjust its attributes, get song recommendations, and build a **spider chart** of your style from the songs you choose.

> Status: course prototype (UW, Fall 2026). Single HTML page, no build step, no dependencies.

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

| Step | What you do | Success | Failure |
|---|---|---|---|
| **① Search for a song** | Type in the search box (it autocompletes), e.g. *"All I Want for Christmas"*. | Sliders animate to that song's values and the song is ringed on the song map. | A **"Song not found"** screen suggests similar titles, or lets you skip and use the sliders directly. |
| **② Tune the attributes** | Drag the vertical sliders (or use the arrow keys / PageUp / PageDown). Uncheck an attribute to ignore it. Adjust **match tolerance**. | **Our recommendations** lists ranked matches (1, 2, 3, …) and the matches light up on the song map. | **"No songs match"**: the previous results stay in place, greyed out, with buttons to widen the tolerance or jump to the closest song. |
| **③ Build your spider chart** | Tick **Add to chart** on any recommendation, or press ▶ to play it (played songs are added automatically). | The spider chart shows the average of your chosen songs. | Before any song is added, an empty state explains what to do. |

Other interactions:

- **Song map** (scatter plot of all 900 songs): hover a dot for details, click a dot to use that song as your baseline, and switch either axis with the X / Y dropdowns.
- **▶ Play** opens Spotify's embedded player for the track (a preview, or the full song if you are logged in to Spotify).
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

---

## Project structure

```
.
├── style-explorer.html        # The whole app: HTML, CSS and JS
├── data/
│   ├── spotify_songs.js       # Trimmed dataset (window.SONG_DATA)
│   └── build_data.py          # CSV → JS converter
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
