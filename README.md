[neon_white_leaderboard_guide(1).md](https://github.com/user-attachments/files/27018595/neon_white_leaderboard_guide.1.md)# How to Scrape Neon White Steam Leaderboards

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

Create a file called `neonwhite_leaderboards.py` in `C:\SteamScraper` and paste in the full script from the bottom of this guide.

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

- **Rate limiting:** The script adds a small delay between batches to be respectful to Steam's servers. Fetching top 1,000 entries across all 125 levels takes approximately 20–25 minutes.
- **Steam must be running:** The script uses the Steamworks SDK locally and requires an active Steam session. It does not use the public Web API and requires no API key.
- **Windows only:** The `steam_api64.dll` approach is Windows-specific. Linux/Mac would require a different DLL and path setup.

---

## How it works

Rather than using the public Steam Web API (which requires a publisher key for leaderboard access), this script loads the game's own `steam_api64.dll` directly via Python's built-in `ctypes` library. It then calls the same Steamworks SDK functions the game itself uses to read leaderboard data — `FindLeaderboard`, `DownloadLeaderboardEntries`, and `GetDownloadedLeaderboardEntry` — polling the async call results until they complete. No third-party libraries are required.
