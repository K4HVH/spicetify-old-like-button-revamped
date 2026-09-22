> **This is a fork of [Maskowh/spicetify-old-like-button-extension](https://github.com/Maskowh/spicetify-old-like-button-extension)**, substantially reworked after the original stopped working following Spotify client updates. The original repository appears inactive — see [Credits](#credits).

# Old Like Button Extension

![Example](example.png)
_Adds the old "like" button back next to the new "Add to Playlist" button on tracklist rows, including a half-heart indicator when you like a different version (remaster, live version, etc.) of the same recording._

## What changed in this fork

The original extension broke because Spotify changed several things it depended on:

- **Liked-status and add/remove had stopped working.** The extension used an internal Cosmos endpoint and the public Spotify Web API (`api.spotify.com`) to read/write Liked Songs. Spotify has since removed that internal endpoint and started rate-limiting `api.spotify.com` heavily, even for authenticated requests from the client itself. This now goes through `Spicetify.Platform.LibraryAPI`, the same API the client uses internally.
- **The half-liked (ISRC) feature was completely broken**, for the same reason — ISRC lookups also went through the now-dead `api.spotify.com`, with no documented replacement. This fork instead calls the same internal metadata service Spotify's own client uses, found via reflection over its webpack module registry, and decodes the response with a small hand-written protobuf parser.
- **The tooltip had no visible styling.** It used `Spicetify.Tippy`, an API that no longer matches how Spotify actually renders tooltips. Replaced with a small tooltip matching the real one's measured appearance.
- **The button was 6px taller than its neighbors**, from a hardcoded CSS class name left over from an older Spotify version.
- **A row could show the wrong track's liked status** after Spotify's virtualized list recycled it for a different song, because the row-identification logic searched in the wrong direction through React's internals. Fixed to walk up the fiber tree from the row instead of down into its children.
- A handful of correctness fixes found through review: a response-handling bug that could permanently hang the refresh loop from a single malformed reply, a missing re-entrancy guard that let rapid double-clicks double-fire, two React root leaks, and a race where a slow periodic refresh could revert a click that had already succeeded.

This is close to a full rewrite of the extension's core logic (see git history for the diff). The visual design and overall approach (playbar + tracklist row insertion) are still the original's.

## Installation

### Manual

1. Install [Spicetify](https://spicetify.app).
2. Run `spicetify config-dir` to find your config folder, then place `oldLikeButton.js` in its `Extensions` subfolder.
3. Run `spicetify config extensions oldLikeButton.js` then `spicetify apply`.

### Marketplace

Search "Old Like Button" in the Extensions tab of Marketplace and install it. Marketplace discovers extensions automatically from public repos tagged `spicetify-extensions` on GitHub, so this fork is picked up the same way the original was.

## Usage

- **Full heart** — this track is in your Liked Songs.
- **Half heart** — you don't have this exact track liked, but you do have a different version of the same recording (same ISRC — e.g. a remaster) liked.
- **No heart** — neither.

Hover a tracklist row to see the button; click to add or remove from Liked Songs.

## Credits

Original extension by [Maskowh](https://github.com/Maskowh), with the rendering approach based on [ohitstom](https://github.com/ohitstom)'s [Quick Queue extension](https://github.com/ohitstom/spicetify-extensions/blob/main/quickQueue/README.md). The original repository appears inactive, which is why this exists as an independent fork rather than a set of pull requests.
