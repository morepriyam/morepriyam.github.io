# morepriyam.github.io: Pulse link test host

A stand-in for `mieweb.github.io` while testing Universal Links / App Links for Pulse pairing ([mieweb/pulse#252](https://github.com/mieweb/pulse/issues/252)). Not used by any shipped build; once the flow is proven, these files move to `mieweb/mieweb.github.io` (association files) and `mieweb/pulse` `pages/` (the `/open` page).

## Files

| Path | What |
|---|---|
| `.nojekyll` | Publish as-is, so `.well-known/` is served. |
| `.well-known/apple-app-site-association` | iOS: `X5873NL7XM.com.mieweb.pulse` handles `/pulse/open*` and `/pulse/watch*`. |
| `.well-known/assetlinks.json` | Android: `com.mieweb.pulse` signed with the two local debug keys (Expo's default `android/app/debug.keystore` and `~/.android/debug.keystore`). |
| `pulse/open.html` | Served at `/pulse/open` (no trailing-slash redirect). The page a pairing link lands on when Pulse is not installed. |

A pairing link looks like:

```
https://morepriyam.github.io/pulse/open#v=1&artifactId=<uuid>&server=<https url>&token=<token>
```

The parameters are in the fragment, so GitHub never receives them.

## What the page does

- **Android:** goes straight to the Play Store (`market://`) with the fragment as `referrer`, once per visit; the badge is the fallback if the browser blocks it.
- **iOS:** "Get Pulse" copies the link (Safari needs a tap for that) and opens the App Store.
- **Both:** "Already have Pulse? Open it" opens `pulsecam://?<fragment>`.
- **Desktop:** a QR of the same link.
- The browser console logs what the page read (never the token), including the referrer length.

## Checks

```sh
# Served without a redirect, and with what content type
curl -sI https://morepriyam.github.io/.well-known/apple-app-site-association
curl -sI https://morepriyam.github.io/.well-known/assetlinks.json

# What Apple's CDN cached (this is what iPhones read)
curl -s https://app-site-association.cdn-apple.com/a/v1/morepriyam.github.io

# What Google's verifier thinks
curl -s "https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://morepriyam.github.io&relation=delegate_permission/common.handle_all_urls"

# On the Android phone, after installing a build with the intent filter
adb shell pm get-app-links com.mieweb.pulse
```
