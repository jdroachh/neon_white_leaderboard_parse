# How to Scrape Neon White Steam Leaderboards

A guide to pulling global leaderboard data for all levels in Neon White using Python and the Steam API — no developer key required.

---

## Prerequisites

- **Windows PC** with Steam installed
- **Neon White** owned and installed on Steam
- **Python 3.8+** installed from [python.org](https://python.org) — do not use the Microsoft Store version

---

## Step 1 — Create your project folder

Create a folder called `C:\SteamScraper`. All files for this project will live here.

---

## Step 2 — Copy the Steam API DLL

The script communicates with Steam via a native DLL that ships with Neon White. Copy it from the game's install directory into your project folder:

**Source:**
```
C:\Program Files (x86)\Steam\steamapps\common\Neon White\Neon White_Data\Plugins\x86_64\steam_api64.dll
```

**Destination:**
```
C:\SteamScraper\steam_api64.dll
```

> If Steam is installed in a different location, adjust the path accordingly. You can find it by right-clicking Neon White in Steam → Manage → Browse local files.

---

## Step 3 — Create the script

Create a file called `neonwhite_leaderboards.py` in `C:\SteamScraper` and paste in the full script from this repo. You can also download the `neonwhite_leaderboards.py` from the repo and move it into the folder `C:\SteamScraper`.

> The script automatically creates `steam_appid.txt` on first run — you do not need to create it manually.

Your folder should now look like this:

```
C:\SteamScraper\
    steam_api64.dll
    neonwhite_leaderboards.py
```

---

## Step 4 — Run the script

1. Open Steam and log in
2. Open Command Prompt (`Win + R` → type `cmd` → Enter)
3. Run the following commands:

```
cd C:\SteamScraper
python neonwhite_leaderboards.py
```

The script will connect to Steam, iterate over all 121 leaderboards, and write results to:

```
C:\SteamScraper\neon_white_top1000.csv
```

Progress is printed to the console as it runs. Results are flushed to disk after each level, so if the script is interrupted you won't lose data already collected.

---

## Output format

The CSV contains the following columns:

| Column | Description |
|---|---|
| `level` | Display name of the level (e.g. "Movement") |
| `rank` | Global rank on the leaderboard |
| `steam_id` | Player's 64-bit Steam ID |
| `name` | Player's Steam display name at time of scrape |
| `score_ms` | Raw score in milliseconds as stored by Steam |
| `time` | Human-readable time in seconds (e.g. `17.442`) |

> **Note on cheaters:** The leaderboards are not fully moderated. Scores that are physically impossible for a given level (e.g. sub-5 seconds on a 17-second world record level) are cheated entries. Filter these out based on known world record times if you need clean data.

> **Note on player names:** Names are resolved at scrape time via the Steam friends API. Players who are not in your friends list will return their public display name. Names can change after the scrape is taken.

---

## Notes and limitations

- **Rate limiting:** The script adds a small delay between batches to be respectful to Steam's servers. Fetching top 1,000 entries across all 121 levels takes approximately 20–25 minutes.
- **Steam must be running:** The script uses the Steamworks SDK locally and requires an active Steam session. It does not use the public Web API and requires no API key.
- **Windows only:** The `steam_api64.dll` approach is Windows-specific. Linux/Mac would require a different DLL and path setup.

---

## Individual Level Search

In addition to the bulk export script, a second script `IL_level_search.py` provides an interactive search tool for querying a single level's leaderboard at a time.

### Setup

Place `IL_level_search.py` in `C:\SteamScraper\` alongside `steam_api64.dll`. No additional setup is required.

```
C:\SteamScraper\
    steam_api64.dll
    neonwhite_leaderboards.py
    IL_level_search.py
```

### Running

With Steam open:

```
cd C:\SteamScraper
python IL_level_search.py
```

### Usage

The script runs as an interactive loop with four prompts:

**1. Level name** — Type any part of a level name and the script will match it against the full list of 121 levels. Typing `list` shows all levels numbered. Partial matches are supported — typing `fire` will match both `Fireball` and `Firecracker` and ask you to choose.

**2. Entry count** — Enter how many entries to fetch (e.g. `10`, `100`, `500`). The total number of entries on that leaderboard is shown first so you know the maximum available.

**3. Output format** — Choose how to handle the results:

| Option | Behaviour |
|---|---|
| `1` | Print results to the console |
| `2` | Save results to a CSV file |
| `3` | Both — print to console and save to CSV |

**4. Search again** — After results are shown, the script asks if you want to search another level. Type `y` to continue or `n` to exit.

### CSV output

When saving to CSV, the file is named automatically based on the level and entry count — for example `Movement_top10.csv` — and saved to `C:\SteamScraper\`. The columns are identical to the bulk export format: `rank`, `steam_id`, `name`, `score_ms`, and `time`.

---

## How it works

Rather than using the public Steam Web API (which requires a publisher key for leaderboard access), this script loads the game's own `steam_api64.dll` directly via Python's built-in `ctypes` library. It then calls the same Steamworks SDK functions the game itself uses to read leaderboard data — `FindLeaderboard`, `DownloadLeaderboardEntries`, and `GetDownloadedLeaderboardEntry` — polling the async call results until they complete. No third-party libraries are required.
