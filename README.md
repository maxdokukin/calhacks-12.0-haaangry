# Haaangry — if TikTok, Tinder and DoorDash had a baby

CalHacks 12.0 hackathon · October 24–26, 2025 · Team: Max Dokukin, Bradley Haraguchi · Status: Completed (hackathon MVP)

[![Watch the demo](https://img.youtube.com/vi/qiALzNWwzLU/hqdefault.jpg)](https://youtube.com/shorts/qiALzNWwzLU)

## Overview

Haaangry is an iOS app that turns short food videos into either a delivery order or a recipe. Users scroll a
full-screen vertical feed of cooking and food-review clips; swiping left asks Claude to search the web for three recipe
articles and three recipe videos that match the clip, and swiping right asks Claude to pick three nearby restaurants and
three dishes each from a 400-item catalog, which the user can order in a mock checkout. A SwiftUI client talks to a
FastAPI backend that serves 226 locally downloaded YouTube Shorts and wraps the Anthropic API. It was built in the
roughly 40 hours of CalHacks 12.0.

## Highlights

- Two Claude-backed flows: `/recipes` runs two parallel `claude-haiku-4-5` calls with Anthropic's web search and web fetch server tools (up to 3 articles + 3 YouTube links); `/recommend` selects exactly 3 restaurants × 3 menu items from `data/restaurants.json`.
- Video catalog built with `yt-dlp`: 96 search topics → 226 enriched records (title, description, tags, likes, comments, captions metadata), ≤ 180 s each, 221 downloaded to local MP4 and served by FastAPI at `/videos`.
- Restaurant catalog of 20 San Jose-area restaurants × 20 menu items (400 items, prices and tags), every restaurant with a website and menu URL.
- TikTok-style playback: a paging `UICollectionView` hosting SwiftUI cells, an `AVPlayer` pool that keeps only the current ±1 videos warm, looping aspect-fill playback, and pager-level left/right swipe recognizers.
- "Online-first, offline-safe" client: every request falls back to bundled JSON fixtures, and confirmed orders are persisted on-device as order history.

## How it works

```
yt-dlp search (96 topics) → enrich metadata → download MP4s → videos.json
                                                   │
iOS feed ── GET /feed ───────────────────────────── FastAPI (StaticFiles /videos)
   ├─ swipe left  → GET  /recipes   → 2 × Claude (web_search + web_fetch) in parallel → 3 READ + 3 WATCH links
   ├─ swipe right → POST /recommend → Claude picks 3 restaurants × 3 items from restaurants.json
   │                                 → ConfirmView (qty, free-delivery toggle, ETA 30 min) → local order history
   └─ Chat / Talk → POST /llm/text | /llm/voice (on-device speech-to-text) → keyword intent → demo restaurants
```

- **`haaangry-backend/app/main.py`** — FastAPI app: feed, recipes, recommendation, order, profile and LLM endpoints; CORS open; mounts the download folder at `/videos`.
- **`haaangry-backend/app/src/ClaudeClient.py`** — Anthropic client (`claude-haiku-4-5`, temperature 0, 1,024 max tokens, 30 s timeout) with JSON-only prompting and optional web tools.
- **`haaangry-backend/data/collect_topics.py`, `download.py`** — YouTube search, metadata enrichment and MP4 download with `yt-dlp`.
- **`HaaangryFrontend/…/Views/VideoFeedView.swift`, `Components/VerticalPager.swift`, `Utilities/PlayerPool.swift`** — the vertical feed, swipe routing and player reuse.
- **`HaaangryFrontend/…/Views/Recipes`, `Views/Restaurant`, `Views/LLM`, `Views/Profile`** — recipe links, recommendations and checkout, chat/voice ordering, profile with order history.
- **`HaaangryFrontend/…/Networking/APIClient.swift`** — async/await client with fixture fallback.

What is mocked in the MVP: `/llm/text` and `/llm/voice` use keyword rules over a three-restaurant stub (`mock_data.py`), not an LLM;
orders, delivery, payments and the profile are simulated.

<img src="doc/slides%20pic/slide%208.png" alt="App screens and API routes" width="800">

## Results

| Metric | Value | Source / note |
|---|---|---|
| Video catalog | 226 records (225 unique), 221 playable locally | `data/videos.json` |
| Search topics | 96 (78 returned videos) | `data/collect_topics.py`, `videos.json` |
| Video length | 7–180 s, median 59 s | `videos.json` `duration_seconds` |
| Restaurant catalog | 20 restaurants × 20 items = 400 items | `data/restaurants.json` (603 lines) |
| Website / menu URL coverage | 20 / 20 restaurants | `data/restaurants.json` |
| Claude calls per recipe request | 2, in parallel (articles, videos) | `app/main.py` `_recipes_core` |
| Claude API response time | ~3 s average (self-reported) | DevPost write-up, `doc/CalHacks Blort-2.docx`; not logged in the repo |

The catalog numbers are counted from the data files. The latency figure comes from the team's DevPost write-up; no timing
logs are in the repository.

## Getting started

Backend (Python 3.11 recommended):

```bash
cd haaangry-backend
python -m venv .venv && source .venv/bin/activate
pip install -r requrements.txt          # file name is spelled this way in the repo
export ANTHROPIC_API_KEY="<your key>"   # required for /recipes and /recommend
(cd data && python collect_topics.py && python download.py)   # writes metadata JSON, downloads MP4s into data/downloads
./run.sh                                # or: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
curl -s http://127.0.0.1:8000/feed | jq length
```

`FEED_JSON` (default `./data/videos.json`) and `DOWNLOAD_DIR` (default in `run.sh`: `./data/downloads`) override the data
paths. `/feed` only returns videos whose MP4 exists in the download folder, so the videos must be downloaded first. They are
not in the repository. `download.py` writes `youtube_video_links_enriched_downloaded.json`; `videos.json` is the feed file
in the same format with `download_path` entries.

iOS app (macOS with Xcode; the project targets iOS 26.0):

```bash
open HaaangryFrontend/HaaangryFrontend.xcodeproj
```

Set `APIBaseURL` in `HaaangryFrontend/HaaangryFrontend/Info.plist` to the backend address (it is set to a hackathon LAN
IP; the simulator fallback is `http://127.0.0.1:8000`), select a simulator or device and run. Without a backend the app
shows bundled fixtures.

## Documents

- [Slides](docs/slides.pdf) (8 slides, CalHacks 12.0, October 26, 2025)
- [Demo video](https://youtube.com/shorts/qiALzNWwzLU)
- [DevPost write-up draft](doc/CalHacks%20Blort-2.docx)
- [Backend README](haaangry-backend/README.md) — full API reference · [Frontend README](HaaangryFrontend/README.md)
- Source repos with full commit history: [backend](https://github.com/maxdokukin/calhacks-12.0-haaangry-backend) · [frontend](https://github.com/maxdokukin/calhacks-12.0-haaangry-frontend)
