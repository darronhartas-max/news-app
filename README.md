# news-app
text only news feed for poor internet connectivity 

# News PWA

A minimal, text-only news reader built for reading headlines on a weak/patchy mobile connection. Installable as a PWA on Android (Chrome → ⋮ → Add to Home screen).

**Live URL:** https://darronhartas-max.github.io/news-app/

## What it does

- Pulls headlines from **Google News RSS** across 8 categories: Top, World, Business, Tech, Sport, Entertainment, Health, Science
- Each headline shows source, title, a short summary (from the RSS description), and a relative timestamp ("2h ago")
- Shows 20 headlines per category with a **"Show more"** button to reveal another 20 from what's already fetched (no extra request)
- Tapping a headline opens the original article in the browser

## Offline / weak-connection behaviour

- Every category's headlines are cached in **localStorage** after a successful fetch
- Opening any tab shows the cached version **instantly**, then quietly fetches a fresh copy in the background and updates in place
- If the live fetch fails (no signal, proxy down, etc.) and a cached copy exists, it keeps showing the cached headlines with a status message showing the actual error
- If there's no cached copy and the fetch fails, it shows a clear error message instead of a blank screen
- Network requests retry up to 2 times with backoff and an 8-second timeout per attempt, so a single dropped packet on a poor connection doesn't immediately fail the whole load

## How fetching works

Google News RSS doesn't send CORS headers, so it can't be fetched directly from a browser. The app routes requests through a small chain of free CORS proxies, tried in order until one works:

1. `api.allorigins.win/get` (JSON-wrapped, follows redirects reliably)
2. `api.codetabs.com/v1/proxy`
3. `api.allorigins.win/raw`

On app open, the current tab loads first, then all other 7 categories are silently prefetched in the background so switching tabs feels instant.

## Refresh behaviour

- The **↻ button** refreshes only the currently visible category (not all tabs)
- A pull-down gesture (≥80px from the top of the list) also triggers a refresh — built manually because standalone/installed PWAs lose Chrome's native pull-to-refresh gesture
- The status line shows `cached` while displaying a saved copy, then updates to `live · updated HH:MM` once a fresh fetch completes
- Each category tracks its own fetch sequence number, so a background prefetch of one tab can never overwrite or block a manual refresh of another

## Known limitations

- Relies on free third-party CORS proxies — if all three are down or rate-limited at once, only cached headlines will show until they recover
- Google News RSS itself doesn't update continuously, so refreshing moments after a previous fetch may return identical headlines (this is expected, not a bug)
- GitHub Pages is served via a CDN that can take up to an hour to reflect a new deploy; the version number in the title/header (currently v6) is there specifically to confirm which version is actually loaded
- Hosted as a public GitHub repo (`news-app`) — fine for this use case, but anything sensitive shouldn't be added to it

## Files

- `index.html` — the entire app (HTML/CSS/JS in one file)
- `manifest.json` — PWA manifest (name, icons, standalone display mode, start URL)

