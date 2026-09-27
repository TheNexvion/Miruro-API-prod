<div align="center">
  <img src="https://www.miruro.to/icon-512x512.png" alt="Miruro API" width="150" style="border-radius: 20%; box-shadow: 0 0 20px rgba(56, 189, 248, 0.5);">
  <br><br>
  
  # Miruro API v2.0
  
  **The ultimate, decrypted, and fully reverse-engineered native Python backend for Miruro.**
  
  [https://github.com/walterwhite-69/Miruro-API](https://github.com/walterwhite-69/Miruro-API)
</div>

<br>

---

## What This Does

Miruro's frontend communicates with its backend through a `secure/pipe` tunnel that base64-encodes, gzip-compresses, and encrypts every request. This project bypasses all of that and gives you simple, direct REST endpoints to:

1. **Search & filter** anime with full AniList metadata
2. **Get complete anime info** — characters, staff, relations, recommendations, trailer, stats, and all metadata in one request
3. **Browse collections** — trending, popular, upcoming, recent, schedule, and spotlight — all paginated
4. **List episodes** with decoded episode IDs from multiple providers
5. **Get M3U8 streaming URLs** for any episode
6. **Autocomplete** search suggestions for dropdown UIs

No headless browsers, no Selenium — just lightweight async HTTP requests.

<br>

## All Endpoints

### 🔍 Search & Discovery

| Endpoint | Description | Params |
|---|---|---|
| `GET /search?query={name}` | Full-text anime search with rich metadata (20+ fields per result) | `query` (required), `page`=1, `per_page`=20 |
| `GET /suggestions?query={name}` | Lightweight autocomplete for dropdowns — returns id, title, poster, format, status, year. Max 8 results. | `query` (required) |
| `GET /filter` | Advanced browse/filter by any combination of genre, tag, year, season, format, status, sort | All optional — see below |

#### Filter Parameters

| Param | Values |
|---|---|
| `genre` | Action, Romance, Comedy, Drama, Fantasy, Sci-Fi, etc. |
| `tag` | Isekai, Time Skip, Reincarnation, etc. |
| `year` | 2025, 2024, etc. |
| `season` | WINTER · SPRING · SUMMER · FALL |
| `format` | TV · MOVIE · OVA · ONA · SPECIAL |
| `status` | RELEASING · FINISHED · NOT_YET_RELEASED · CANCELLED |
| `sort` | SCORE_DESC · POPULARITY_DESC · TRENDING_DESC · START_DATE_DESC |
| `page` / `per_page` | Pagination (defaults: 1 / 20, max per_page: 50) |

---

### 📊 Collections (All Paginated)

| Endpoint | Description |
|---|---|
| `GET /trending` | Currently trending anime |
| `GET /popular` | Most popular anime of all time |
| `GET /upcoming` | Most anticipated upcoming anime |
| `GET /recent` | Currently airing / this season's anime |
| `GET /spotlight` | Curated "What's Hot" list (trending + popular) |
| `GET /schedule` | Airing schedule for the next week |

All collection endpoints accept `page` and `per_page` query params and return:

```json
{
  "page": 1,
  "perPage": 20,
  "total": 5000,
  "hasNextPage": true,
  "results": [ ... ]
}
```

Each anime in `results` includes 20+ fields: title (romaji/english/native), coverImage, bannerImage, format, season, seasonYear, episodes, duration, status, averageScore, meanScore, popularity, favourites, genres, source, countryOfOrigin, studios, nextAiringEpisode, startDate, endDate, and more.

---

### 📖 Anime Details

| Endpoint | Description |
|---|---|
| `GET /info/{anilist_id}` | **Complete anime page** — everything in one request |
| `GET /anime/{id}/characters` | Paginated character list with voice actors |
| `GET /anime/{id}/relations` | All related media (sequels, prequels, side stories, spin-offs) |
| `GET /anime/{id}/recommendations` | Community recommendations sorted by rating |

#### What `/info/{id}` Returns

Everything you need to build a full anime detail page:

- **Core**: id, idMal, title (romaji/english/native), description, coverImage, bannerImage
- **Metadata**: format, season, seasonYear, episodes, duration, status, source, countryOfOrigin
- **Scores**: averageScore, meanScore, popularity, favourites, trending
- **Taxonomy**: genres, tags (with rank & spoiler flag), synonyms, hashtag
- **People**: characters (25, with voice actors), staff (25, with roles)
- **Related**: relations (sequels/prequels/etc.), recommendations (10, with ratings)
- **Media**: trailer (YouTube/Dailymotion), streamingEpisodes, externalLinks
- **Stats**: scoreDistribution, statusDistribution
- **Studios**: name, isAnimationStudio, siteUrl
- **Dates**: startDate, endDate, nextAiringEpisode
- **Links**: siteUrl, externalLinks (MAL, official site, etc.)

---

### ▶️ Streaming (3-Step Flow)

To get a video stream, follow these 3 steps in order:

#### Step 1: Get Episodes — `GET /episodes/{anilist_id}`

Returns all episodes from multiple providers (kiwi, arc, zoro, jet, etc.) organized by audio type.

```json
{
  "mappings": { "anilistId": 178005, "malId": 56885, "kitsuId": ... },
  "providers": {
    "kiwi": {
      "episodes": {
        "sub": [
          {
            "id": "watch/kiwi/178005/sub/animepahe-1",
            "number": 1,
            "title": "Episode Title",
            "image": "https://serveproxy.com/url?url=...",
            "airDate": "2026-01-04",
            "duration": 1420,
            "description": "...",
            "filler": false
          }
        ],
        "dub": [ ... ]
      }
    },
    "arc": { ... },
    "zoro": { ... }
  }
}
```

#### Step 2: Get Sources [SUPER SIMPLE]

Just take the direct `id` from the Step 1 response and use it as the URL. No manual parameters or complex IDs needed!

**Endpoint:** `GET /{id}`
**Example:** `GET /watch/kiwi/178005/sub/animepahe-1`

```json
{
  "streams": [
    { "url": "https://.../master.m3u8", "type": "hls", "quality": "1080p" }
  ],
  "subtitles": [
    { "file": "https://...", "label": "English", "kind": "captions" }
  ],
  "intro": { "start": 0, "end": 90 },
  "outro": { "start": 1300, "end": 1420 }
}
```

> [!TIP]
> This endpoint automatically handles decryption, provider selection, and category matching. It returns the direct M3U8/HLS streaming URL and intro/outro timestamps.

<details>
<summary><b>Fallback / Detailed Option</b></summary>
If you need manual control, you can use the traditional endpoint:
`GET /sources?episodeId=...&provider=...&anilistId=...&category=...`
</details>

#### Step 3: Play

Feed `streams[0].url` into any HLS player (Video.js, hls.js, VLC, mpv). Subtitles are either **hard-subbed** (baked into the video for kiwi/pahe) or provided in the `subtitles` array (VTT links for zoro/arc). Use `intro`/`outro` timestamps for skip buttons.

<br>

## Setup

```bash
git clone https://github.com/walterwhite-69/Miruro-API.git
cd Miruro-API
pip install -r requirements.txt  
uvicorn api:app --host 0.0.0.0 --port 8000
```

Then open `http://localhost:8000/` for interactive API docs.

<br>

## Disclaimer

This project is for educational purposes and API integrity research only. The author takes absolutely zero responsibility for network usage. Code contains zero skiddable artifacts.

<br>

**Author:** Walter | **GitHub:** [walterwhite-69](https://github.com/walterwhite-69)


## Production API service

The service is a stateless FastAPI application and is suitable for Render, Docker, or another HTTPS platform. It exposes JSON API routes plus the standard FastAPI documentation pages.

### Base URL

The public base URL is `https://miruro-api-prod-us.onrender.com`. The health check is `GET /health`.

### Endpoints

- `GET /health`
- `GET /search?query={name}&page=1&per_page=20`
- `GET /suggestions?query={name}`
- `GET /filter` with optional `genre`, `tag`, `year`, `season`, `format`, `status`, `sort`, `page`, and `per_page`
- `GET /spotlight`, `/trending`, `/popular`, `/upcoming`, `/recent`, `/schedule` with `page` and `per_page`
- `GET /info/{anilist_id}`
- `GET /anime/{anilist_id}/characters?page=1&per_page=25`
- `GET /anime/{anilist_id}/relations`
- `GET /anime/{anilist_id}/recommendations?page=1&per_page=10`
- `GET /episodes/{anilist_id}`
- `GET /sources?episodeId=...&provider=...&anilistId=...&category=sub`
- `GET /watch/{provider}/{anilist_id}/{category}/{slug}`

Example:

```bash
curl https://miruro-api-prod-us.onrender.com/health
curl 'https://miruro-api-prod-us.onrender.com/search?query=one%20piece'
```

Errors use JSON responses such as `{ "error": "Route not found" }`, `{ "error": "Invalid request", "details": [...] }`, `429` for rate limiting, `502` for upstream provider failures, and `504` for upstream timeouts.

### Configuration

All environment variables are optional:

| Variable | Default | Purpose |
|---|---:|---|
| `ALLOWED_ORIGINS` | `*` | Comma-separated browser origins. Credentials are disabled for safe public CORS. |
| `RATE_LIMIT_PER_MINUTE` | `120` | Per-process limit per client IP and route. |
| `CACHE_TTL_SECONDS` | `30` | In-memory cache TTL for safe metadata and episode responses. |
| `API_KEY` | unset | Optional `X-API-Key` value for clients that need an additional shared-secret gate. |

### Local production-style run

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
PORT=8000 .venv/bin/uvicorn api:app --host 0.0.0.0 --port 8000 --proxy-headers
```

### Container run

```bash
docker build -t miruro-api .
docker run --rm -p 8000:8000 miruro-api
```

The platform must route its HTTPS health check to `/health`, set `PORT` when required, and provide the optional variables above through its secret/environment configuration rather than committing a `.env` file.
