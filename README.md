# MLDS 422 Homework 5: Ariana Grande Music Data with APIs

This project collects track data and lyrics for five Ariana Grande studio albums from two APIs, combines them into one dataset, and explores two insights about her music.

**Albums (66 songs):** Yours Truly (2013), Sweetener (2018), thank u, next (2019), Positions (2020), eternal sunshine (2024)

## APIs used

- **Last.fm API (Part 1):** album track lists, track number, song duration, and each song's listeners and playcount. I chose Last.fm instead of Spotify because it only requires a free API key.
- **Genius API (Part 2):** album, release date, pageviews, featured artists, writers, producers, annotation count and pyongs count for each song. The Genius API does not return lyrics, so the lyrics are taken from each song's genius.com page.

## Project structure

```
hw5/
├── README.md
├── requirements.txt
├── data/
│   ├── lastfm_tracks.csv      # Part 1 output
│   ├── genius_tracks.csv      # Part 2 output
│   └── combined_tracks.csv    # Part 3 output (combined dataset)
└── notebooks/
    ├── 01_lastfm_pull.ipynb   # Part 1: Last.fm pull
    ├── 02_genius_pull.ipynb   # Part 2: Genius pull
    └── 03_combine_eda.ipynb   # Part 3: combine data and EDA
```

## Setup

1. Create and activate a virtual environment (the project was run with Python 3.9):

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   ```

2. Install the packages:

   ```bash
   pip install -r requirements.txt
   ```

3. Create a file named `.env` in the project's top-level folder (next to this README) with your own keys:

   ```
   LASTFM_API_KEY=your_lastfm_api_key
   GENIUS_ACCESS_TOKEN=your_genius_client_access_token
   ```

   - **Last.fm:** create an API account at https://www.last.fm/api to get an API key.
   - **Genius:** create an API client at https://genius.com/api-clients, then click "Generate Access Token" and use the Client Access Token.

   The `.env` file is not included in the submission because it contains private keys. The notebooks load the keys from it with `python-dotenv`, so no keys appear in the code.

## How to run

Run the notebooks in the `notebooks/` folder in this order. Each notebook reads the previous step's output from `data/`.

1. `01_lastfm_pull.ipynb` → saves `data/lastfm_tracks.csv`
2. `02_genius_pull.ipynb` → saves `data/genius_tracks.csv` (uses the song list from Part 1)
3. `03_combine_eda.ipynb` → saves `data/combined_tracks.csv` and contains the EDA

Notebook 2 downloads every song's lyrics page, so it takes several minutes.

## Data files

| File | Rows | Contents |
|---|---|---|
| `lastfm_tracks.csv` | 66 | album, song name, track number, duration (seconds), listeners, playcount |
| `genius_tracks.csv` | 66 | album, song name, lyrics, Genius song ID, release date, pageviews, featured artists, writers, producers, annotation count, pyongs count |
| `combined_tracks.csv` | 66 | both datasets joined on album and song name, with source-specific columns renamed (for example `lastfm_playcount`, `genius_pageviews`) |

## Insights (Part 3)

1. **eternal sunshine has the highest replay value, and song length and lyric repetition don't explain it.** Using Last.fm plays per listener, eternal sunshine's typical song is played about 21 times per listener, compared with 9.7 to 14.9 for the other albums, and it holds 9 of the 10 most-replayed songs. Within the same album, song length has almost no relationship with replays, and songs with more varied lyrics are replayed slightly more.
2. **Ariana's lyrics shift from focusing on "you" to focusing on "me" over her career.** Using Genius lyrics, the share of "I/me" words compared with "you" words is lowest on Sweetener and highest on eternal sunshine, where even the less self-focused songs lean toward "me."

## Notes on data issues and decisions

- **Last.fm corrections:** Last.fm's album data listed "You'll Never Know" as a live version and spelled "thank u, next" as "thank you, next," so both names were corrected before looking up song details. Five missing durations were filled in by hand from Apple Music.
- **Genius matching:** `search_album` was blocked by genius.com (403 error), so each song is searched individually with the official API. A few songs matched the wrong version (for example a live recording or a different artist's song), and Genius's search results can change between runs, so those songs use Genius song IDs chosen by hand. 
- **Lyrics requests:** genius.com blocks standard Python requests, so lyrics pages are downloaded with `curl_cffi`. Requests sometimes time out, so the lyrics function retries before giving up. If many requests fail, try a different network (shared university networks are more likely to be blocked).
- **Joining the datasets:** song titles differ slightly between the two sources (curly vs straight apostrophes, capitalization, labels like "(Remix)"), so the join uses normalized versions of the album and song names. All 66 songs matched.
- A harmless `NotOpenSSLWarning` from urllib3 may appear on macOS with Python 3.9. It is hidden in the notebooks and does not affect the results.