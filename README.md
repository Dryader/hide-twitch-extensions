# Hide Twitch Extensions

A small uBlock Origin filter list that turns Twitch channel **extensions** off for viewers — the leaderboards, drop trackers, sponsor overlays, sub-goal bars and Prime reminders that sit on top of the video — and touches nothing else.

Twitch gives viewers no switch for this. Its own help documentation states that player extensions can be *minimised* but not disabled, because that choice belongs to the streamer. A content blocker is the only viewer-side option.

Unlike the alternatives, this list is **extensions only**: no ads, no channel points, no bits button, no chat changes, no front-page surgery.

---

## Install

### Option A — one click (least setup)

[![Subscribe in uBlock Origin](https://img.shields.io/badge/Subscribe-uBlock%20Origin%20%2F%20AdGuard%20%2F%20ABP-800000?style=for-the-badge)](https://subscribe.adblockplus.org/?location=https%3A%2F%2Fraw.githubusercontent.com%2FDryader%2Fhide-twitch-extensions%2Fmain%2Ffilterlist.txt&title=Hide%20Twitch%20Extensions)

Click the button, then click **Subscribe** on the page that opens. If your browser refuses to open the link, right-click it and choose **"Subscribe to filter list…"** from the uBlock Origin context menu (uBO 1.36+) — that route works from any site.

No dashboard navigation, no copy/paste, and updates arrive on their own: uBlock Origin re-checks this list on the `! Expires: 7 days` header in its own header block (uBO falls back to 5 days for lists that declare nothing).

### Option B — copy/paste the rules (no subscription)

uBlock Origin dashboard → **My filters** → paste the block below → **Apply changes** → reload Twitch. Nothing depends on this repo afterwards, but you won't get fixes when Twitch renames something — you'd come back and re-paste.

```
! Twitch extensions off (viewer side)
||ext-twitch.tv^$subdocument
twitch.tv##[class*="extensions-dock"]
twitch.tv##.extensions-video-overlay-size-container
twitch.tv##.extension-taskbar
twitch.tv##.extension-taskbar-card-container
twitch.tv##.extension-view
twitch.tv##.extension-frame-wrapper
twitch.tv##.extensions-notifications
```

---

## What each rule does

| Rule | Type | Job |
|---|---|---|
| `\|\|ext-twitch.tv^$subdocument` | network | Blocks the iframes every extension renders in (`<id>.ext-twitch.tv`, orchestrated by `supervisor.ext-twitch.tv`). Extensions never load at all. Matches on hostname, so hashed CSS class names and front-end rewrites can't break it. |
| `[class*="extensions-dock"]` | cosmetic | The dock/card UI Twitch draws for extensions. |
| `.extensions-video-overlay-size-container` | cosmetic | The invisible sizing box over the video. Also a known cause of player controls going unclickable and the cursor refusing to auto-hide. |
| `.extension-taskbar`, `.extension-taskbar-card-container` | cosmetic | The strip of extension buttons under/over the player. |
| `.extension-view`, `.extension-frame-wrapper` | cosmetic | Wrappers left behind once the frame is blocked, so they don't reserve empty space. |
| `.extensions-notifications` | cosmetic | Extension notification popups. |

The network rule is the load-bearing one. Everything else is tidying up.

## What this deliberately breaks

Extensions you might actually want die too — sub-goal bars, song-request panels, map/stat trackers, the Prime loot reminder, interactive game overlays. There is no per-extension allowlist here by design.

To get them back for one channel: click the uBlock Origin icon and hit the big power button (disables filtering on twitch.tv for that site), then reload. Toggle it back on when you're done.

## What it does not touch

Ads (this is not an ad blocker), channel points, bits, the share button, Prime offers, the front-page carousel, chat, badges, emotes, the chat leaderboard header, and panels below the stream that aren't extensions.

## Browser support

| Browser | Works? | Notes |
|---|---|---|
| Firefox + uBlock Origin | Yes | Recommended. Full uBO, custom filters supported. |
| Firefox forks (LibreWolf etc.) | Yes | Same engine. |
| Brave | Yes, different UI | Paste the same rules into `brave://adblock` → custom filters. |
| Chrome / Edge + uBO **Lite** | No | Manifest V3 uBOL has no "My filters" equivalent for these rules. Use Firefox or Brave. |
| AdGuard | Probably | ABP-compatible syntax; the optional `#?#` procedural rule is uBO/AdGuard only. Untested here. |

## Verify it worked

1. Open a channel that has a visible overlay extension (leaderboard, drop tracker).
2. Open the uBlock Origin **logger**, reload the page.
3. You should see `ext-twitch.tv` entries of type `subdocument` in red (blocked).

If the extension is gone but a blank gap remains, that's a missing cosmetic rule, not a failed block — see below.

## If something leaks through

Twitch renames CSS classes regularly. Don't guess at selectors:

1. Right-click the leftover element → **Block element** (uBlock Origin's picker).
2. The picker generates the correct *current* selector for you.
3. Open an issue with that selector and it goes into the list.

## Verification status

Honest about what has and hasn't been checked, because filter lists rot silently.

- **2026-08-30 — network rule verified live.** `https://supervisor.ext-twitch.tv/supervisor/v1/index.html` returns HTTP 200 `text/html`, confirming the extension host frame is still in use. Per-extension frames still load from `<32-char-id>.ext-twitch.tv/.../component.html?anchor=component…`.
- **2026-08-30 — cosmetic selectors corroborated, not live-DOM-tested.** `extension-taskbar`, `extensions-dock__layout` and `extensions-video-overlay-size-container` appear in a 2018 filter list and `extension-taskbar-card-container` / `extensions-dock__layout` in a list maintained through 2024+. Two independent sources six years apart agreeing is good evidence the names are stable, but they have not been re-checked against a live channel DOM in this release. Report leftovers and they'll be fixed with picker-generated selectors.
- **Twitch's no-disable stance** is from Twitch's extensions help documentation for viewers ("can be minimized but not disabled by a viewer"), as quoted in community write-ups; the help page itself was not directly reachable when this was written.

## Prior art, and why this exists

| Project | Hides extensions? | State |
|---|---|---|
| `ultramegatom/adblock-twitch-garbage` | Yes | Filter file last modified **October 2018**. Cosmetic-only (extensions still load). Also strips channel points, bits, share button, Prime offers, GDPR banner, front-page carousel. |
| `DandelionSprout/adfilt` → *Twitch: Pure Viewing Experience* | No | Jan 2024 beta, zero extension rules. Its *Anti-Amazon List for Twitch* has surgical rules for specific Prime extensions only. |
| `BevizLaszlo/UBlock-Filters-for-Social-Media` | No | Actively maintained, nice list — but its Twitch coverage is home feed + recommended channels. |
| `pixeltris/TwitchAdSolutions` | No (ads) | Archived 2026-03-05. |
| Browser add-ons (Previews, "Twitch: Hide Elements") | Yes | Works, but means installing an extension with access to your Twitch data. "Twitch: Hide Elements" was last updated Dec 2022. |
| Assorted gists | Partially | Mostly dead selectors (`#js-player-extension-root`) or hashed React class names that break on every deploy. |

Nothing current does "all Twitch extensions off, and only that". Hence this.

## Uninstall

Remove the imported list under **Filter lists → Custom**, or delete the lines from **My filters**, then reload Twitch. Nothing is changed on Twitch's side, ever — this only changes what your browser draws.

## License

MIT. See [LICENSE](LICENSE).
