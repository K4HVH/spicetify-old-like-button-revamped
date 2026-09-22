> **This is a fork of [Maskowh/spicetify-old-like-button-extension](https://github.com/Maskowh/spicetify-old-like-button-extension)**, substantially reworked after the original stopped working following Spotify client updates. The original repository appears inactive (see [Credits](#credits)).

# Old Like Button Extension

![Example](example.png)
_Adds the old "like" button back next to the new "Add to Playlist" button on tracklist rows, including a half-heart indicator when you like a different version (remaster, live version, etc.) of the same recording._

## What changed in this fork

The original extension broke because Spotify changed several things it depended on.

The core problem is `api.spotify.com`: the internal Cosmos endpoint and the public Web API this extension used to read and write Liked Songs are both gone (Spotify removed the internal one and now rate-limits the public one heavily, even for authenticated requests from the client itself). Liked-status display, add/remove, and the ISRC-based half-liked feature all broke for this reason. Liked-song state now goes through `Spicetify.Platform.LibraryAPI`; ISRC lookups go through Spotify's own internal metadata service instead, found by reflecting over its webpack module registry and decoded with a small hand-written protobuf parser, since there's no documented replacement for what `api.spotify.com` used to provide.

Everything else found along the way:

- A row could show the wrong track's liked status after Spotify's virtualized list recycled it for a different song. The row-identification logic was searching the wrong direction through React's internals; it now walks up the fiber tree from the row instead of down into its children.
- The tooltip used `Spicetify.Tippy`, an API that no longer matches how Spotify renders its own tooltips, so it showed with no visible styling. Replaced with a small tooltip matching the real one's measured appearance.
- The button was 6px taller than its neighbors, from a hardcoded CSS class name left over from an older Spotify version.
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

- **Full heart**: this track is in your Liked Songs.
- **Half heart**: you don't have this exact track liked, but you do have a different version of the same recording liked (same ISRC, e.g. a remaster).
- **No heart**: neither.

Hover a tracklist row to see the button; click to add or remove from Liked Songs.

## Credits

Original extension by [Maskowh](https://github.com/Maskowh), with the rendering approach based on [ohitstom](https://github.com/ohitstom)'s [Quick Queue extension](https://github.com/ohitstom/spicetify-extensions/blob/main/quickQueue/README.md). The original repository appears inactive, which is why this exists as an independent fork rather than a set of pull requests.
