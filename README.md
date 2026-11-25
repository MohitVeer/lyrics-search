## LyricsSearch

A small search app for songs and artists backed by the [lyrics.ovh](https://lyrics.ovh) API — search by name, browse paginated results, and pull up the full lyrics for anything in the list.

### Overview

Typing a search term into the form and submitting it queries the API's suggest endpoint and lists matching songs with the artist name, title, and a "Get Lyrics" button. If the API returns more results than fit on one page, Prev/Next buttons appear to page through them. Clicking "Get Lyrics" on any result fetches the actual lyrics text and swaps the result list out for it.

### Why I Built It

This one is mostly about consuming a third-party JSON API with plain `fetch` calls and rendering results dynamically, including a pagination flow driven by URLs the API itself hands back (rather than page numbers I'd track myself). It also covers event delegation — the lyrics buttons are created dynamically after a search, so their click handling has to be attached to a static parent rather than to each button individually.

### Features

- Search by artist or song name, submitted via a form
- Dynamically rendered result list with artist, title, and a lyrics button per row
- Prev/Next pagination using the URLs returned directly by the API
- Full lyrics fetch and display for a selected song, with newlines converted to line breaks
- Basic error handling: empty search term is blocked with an alert, and failed network requests show an inline message instead of failing silently

### JavaScript Concepts

- **Async/await with `fetch`**: `searchSongs`, `getMoreSongs`, and `getLyrics` all use `async`/`await`, with the first two wrapped in try/catch and an `res.ok` check so a network failure surfaces to the user instead of doing nothing.
- **Event delegation**: rather than attaching a click listener to every "Get Lyrics" button (which don't exist until a search runs), a single listener on the `#result` container checks `e.target.tagName` to catch clicks on any button rendered inside it.
- **Dynamic HTML generation**: `showData` builds the result list with `Array.prototype.map`/`join`, embedding `data-artist`/`data-songtitle` attributes read back out via `getAttribute` when a lyrics button is clicked.
- **`encodeURIComponent`**: search terms and artist/song names are URI-encoded before being interpolated into request URLs, so special characters like `&`, `#`, or spaces don't break the request.
- **Regex replace**: `getLyrics` swaps `\r\n`/`\n` line endings in the raw lyrics text for `<br>` tags before rendering.

### How It Works

The suggest endpoint (`/suggest/:term`) returns a `data` array of matches plus optional `prev`/`next` URLs for pagination, which `showData` renders directly into inline `onclick="getMoreSongs('...')"` handlers — meaning `getMoreSongs` stays a plain global function so that markup can call it after every re-render. Fetching lyrics hits `/v1/:artist/:title` and checks for the API's own `error` field, since a "not found" case comes back as a normal JSON response rather than an HTTP error.

### How to Run

Open `index.html` in a browser, or serve the folder with a static file server. Requires network access to `api.lyrics.ovh`.

### Technologies

HTML, CSS, vanilla JavaScript, Fetch API.

### What I Learned

The pagination links from the API are full URLs, and the initial version routed them through `cors-anywhere.herokuapp.com` — a public demo CORS proxy — before fetching, on the assumption the API needed it. That proxy has been dead since Heroku discontinued free dynos in late 2022, so Prev/Next never worked. Since the very first search request already fetches `api.lyrics.ovh` directly with no proxy and works fine, the fix was just to drop the proxy prefix from pagination requests too. It's a reminder to check whether a workaround (the proxy) is still doing anything useful before copying it into new code.

### Future Improvements

- Loading indicators while a search or lyrics fetch is in flight
- Debounced live search instead of only searching on form submit
- A proper "no results found" state instead of an empty list
