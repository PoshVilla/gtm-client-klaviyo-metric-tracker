# GTM Client - Klaviyo Metric Tracker

## TLDR;

Google Tag Manager template for tracking Klaviyo events client-side using _learnq. Identify users and send custom metrics without writing code or using custom HTML tags.

## A little more info...

A Google Tag Manager tag template for sending custom events (metrics) to Klaviyo on the **client-side**, without writing any JavaScript.

This template lets you track custom user interactions—such as product views, quiz completions, signups, or any other meaningful activity—and pass event data to Klaviyo for segmentation, flow logic, or reporting. It also handles user identification via email or the Klaviyo cookie (`_kx`), and supports optional properties, event value fields, and profile updates.

---

## 🔄 Changes in This Version

**Last updated:** 23 July 2026

### Non-Technical Summary

This update makes the tag more trustworthy in two ways.

First, **True/False (boolean) values are now handled correctly everywhere**. Previously, a property like "Is Subscriber: false" risked being silently turned into the number `0` behind the scenes — which meant "false" and "unset" could look the same to Klaviyo. That's fixed, so boolean properties now say exactly what they mean, whether you're setting them on an event or on a person's profile.

Second, **the "don't track this more than once every X hours" option now actually works as intended**. Previously it relied on a trick that, on investigation, never really stopped duplicate events from reaching Klaviyo. It's been replaced with a proper mechanism: the tag remembers (via a small browser cookie) the last time each event fired, and skips re-firing it within your chosen time window — the same way it always should have.

There's also a small but important fix: the "_kx Cookie (Exchange Id)" field had an internal naming mismatch that meant, in the actual tracking script, whatever you typed into that field was silently ignored — it never reached Klaviyo. That's now corrected, so identifying visitors by their `_kx` cookie actually works.

Two of these three changes don't require you to reconfigure existing tags. **The Exchange ID fix does**: because of how the mismatch is corrected (see Technical Summary), any tag that already had a value in the "_kx Cookie (Exchange Id)" field will need that value re-entered after updating to this version — the field will otherwise appear blank. You'll also now see a **Boolean** option in the Property Type dropdown that wasn't there before.

### Technical Summary

- **Added `coerceBool()` helper** — safely interprets `true`/`false` (or the strings `"true"`/`"false"`), instead of relying on `value * 1`, which previously coerced `false` to `0` and silently corrupted boolean data.
- **Added `isEmpty()` helper** — a single, consistent check for `null`, `undefined`, and blank-string values, applied before building both event and profile properties. This replaces ad hoc checks and ensures empty values are never sent to Klaviyo's API.
- **Property building is now type-aware**: each property's declared `propertyType` (`text`, `number`, `boolean`, or `array` for events) determines how its value is coerced, with boolean values explicitly bypassing the numeric coercion path that would otherwise corrupt `false`.
- **Added a `Boolean` option to the Property Type dropdown** for both Metric/Event Properties and Profile Properties, so this type-aware handling is actually reachable from the tag's UI (previously the code path existed but had no way to be triggered).
- **Replaced the `$event_id`-based volume limiting with a real cookie-based mechanism.** The previous approach appended a time-bucketed `$event_id` to the event payload, but this did not actually prevent `_learnq.track()` from re-firing — it was dead weight. Volume limiting now reads/writes a `kmt-log` browser cookie (1-year max-age, domain-scoped to the top two labels of the hostname, e.g. `.corporatefinanceinstitute.com`) storing a `eventKey:unixSeconds` pair per event. If the same event fires again inside the configured window, the tag skips the `_learnq.track()` call entirely (while still resolving `gtmOnSuccess()` so GTM doesn't report an error).
- **Added debug logging for the volume-limit gate** — when Debug mode is enabled, the tag now logs when the limit is active, when a cookie is created, and when/why an event is skipped, making it much easier to tell "volume limited" apart from "actually failed" during testing.
- **Removed the dead `$event_id` logic** entirely, along with the debug logging tied to it, to keep the code path simple and honest about what it's doing.
- **New required GTM permissions:** the cookie-based approach calls `getCookieValues`, `setCookie`, and `getUrl`, none of which the previous version needed. The permissions block in this `.tpl` now includes the exact grants GTM requires (see the checklist below) — no manual guessing needed.
- **Fixed the Exchange ID parameter name.** The template parameter was named `echangeId` (typo) while the script read `data.exchangeId` — meaning the "_kx Cookie (Exchange Id)" field was never actually wired up to the tracking logic; any value entered there was silently dropped. The parameter is now correctly named `exchangeId`, matching both the script and the repo's own `sample-klaviyo-tag.json`. **Action required:** any tag already using this field will show it as blank after updating and needs the value re-entered (e.g. `{{Cookie - _kx}}`).
- **Added a `debug` checkbox to the tag UI.** The script has always read `data.debug`, but no corresponding parameter existed in `TEMPLATE_PARAMETERS` — there was no way to turn Debug mode on from GTM's tag editor at all; it only worked if `debug` was set directly via the API or a raw tag import (as the sample tag JSON in this repo does). Added an "Enable debug mode?" checkbox so it's actually usable from the tag UI.

### ✅ Post-Update Checklist (Existing Installations)

If you're updating an already-installed copy of this template (rather than importing fresh), work through this after importing and before publishing:

- [ ] **Grant the `get_cookies` permission** — Cookie access: *Specific*, Allowed cookie names: `kmt-log`
- [ ] **Grant the `set_cookies` permission** — Cookie name: `kmt-log`, Domain: `*`, Path: `*`, Secure: *Any*, Session: *Any*
- [ ] **Grant the `get_url` permission** — URL parts: *Any*, Queries allowed: *Any*
- [ ] **Re-enter the "_kx Cookie (Exchange Id)" value** (e.g. `{{Cookie - _kx}}`) on every existing tag using this template — it will show as blank due to the parameter rename above
- [ ] **Test in Preview mode** before publishing — confirm existing tags still fire correctly and the `kmt-log` cookie is being set
- [ ] **Publish a new container version** — saving the template only updates your workspace draft

This template's `.tpl` file already has the permissions above baked in (see [Updating This Template](#-updating-this-template) below), so if you import the file directly you should only need the last three steps.

---

## ✅ Features

- No need for custom HTML tags
- Identify users via email or `$exchange_id` (`_kx` cookie)
- Send event name, `value`, and unlimited custom event properties
- Optionally include **Profile Properties** (for updating user-level fields)
- Optionally enable **Event Volume Limit** (prevent duplicate tracking per time window, backed by a real browser cookie)
- Supports typed properties: text, number, boolean, or array (comma-separated) for events; text, number, or boolean for Profile Properties (array support for Profile Properties is planned — see [Future Plans](#-future-plans))
- Boolean values are handled safely — `false` is never silently corrupted into `0`
- Built-in debug mode (logs to console + `dataLayer`)
- Fully sandbox-safe for use in GTM Templates
- Compatible with all GTM web containers

---

## 🧱 How It Works

This template uses Klaviyo’s legacy but fully supported `_learnq` method to push data to the Klaviyo tracking queue. When Klaviyo's JavaScript loads, it processes any queued `identify` or `track` calls automatically.

---

## 🤔 Why use `_learnq` instead of the `klaviyo` object?

Klaviyo's modern `klaviyo` object is built using JavaScript Proxy and async-based methods. While this enables advanced functionality (like promises and method chaining), it is **incompatible with GTM’s custom template sandbox**.

`_learnq`, on the other hand, is a simple global array that:
- ✅ Works reliably with GTM’s `createQueue` method
- ✅ Is fully supported by Klaviyo (per their [official docs](https://developers.klaviyo.com/en/docs/introduction_to_the_klaviyo_object))
- ✅ Gets processed automatically once Klaviyo's JS is loaded

| Approach      | Works in GTM Templates? | Needs Klaviyo JS SDK? | Supports Chaining/Callbacks |
|---------------|--------------------------|------------------------|-----------------------------|
| `_learnq`     | ✅ Yes                   | ✅ Yes (to process queue) | ❌ No                       |
| `klaviyo`     | ❌ No (blocked by sandbox) | ✅ Yes (proxy-based)    | ✅ Yes                      |

This template opts for reliability and GTM compatibility over advanced JS features.

---

## 🚀 How to Use

1. **Import the template into GTM** (via the Template Gallery or manually)
2. Create a new tag using the **Klaviyo Metric Tracker**
3. Fill in:
   - **Event Name** – What you want the metric to be called
   - **Email** – Optional; leave blank to use `_kx` cookie
   - **Value** – Optional numeric value (e.g. price)
   - **Properties** – Add custom data fields for the event
   - **Profile Properties** – (Optional) Add persistent attributes to the user profile
   - **Event Volume Limit** – (Optional) Prevent duplicate events by hour-based window
   - **Enable debug mode?** – (Optional) Logs identify/track calls and volume-limit decisions to the console and `dataLayer`; turn off before publishing to production
4. Add a trigger (e.g. form submission, page view)
5. Preview & publish

---

## 🔧 Updating This Template

If you already have this template installed and want to pull in a newer version of this `.tpl` file, **open your existing "Klaviyo Metric Tracker" custom template in GTM first** — don't create a new one.

1. In GTM, go to **Templates** and open the existing **Klaviyo Metric Tracker** custom template (the one your tags already use)
2. Click the **⋮ (three-dot menu)** in the top right → **Import**
3. Select the updated `.tpl` file — this replaces the Info, Fields, Code, Permissions, Tests, and Notes tabs of this *same* template
4. **Save**

Because you updated the same template (same internal ID) rather than creating a new one, your existing tags keep working — they aren't deleted or unlinked. Two things to double-check afterward, though:

- **New permissions:** if the updated code calls a sandboxed API (like reading/writing cookies) that the previous version didn't need, GTM will show an error on Save until you grant it. Work through any prompted permissions — accept them, or set the specific values called for in that version's [Changes in This Version](#-changes-in-this-version) section.
- **Renamed fields:** if a template parameter's internal name changes between versions (this happens occasionally when fixing bugs — see the Exchange ID fix in this version), any tag that already had a value in that field will show it as blank. That's expected — the tag isn't broken, just that one field needs re-entering.

Saving the template only updates your GTM **workspace** draft. As with any GTM change, it won't take effect for site visitors until you **publish a new container version**.

---

## 🛠 Field Notes

- **Value** is auto-converted to a number with 2 decimal places
- **Array-type** properties should be comma-separated (e.g. `Program,Certification,B2C`) — currently supported for Event Properties only
- **Boolean-type** properties accept `true`/`false` and are preserved as real booleans, not converted to `0`/`1`
- **Exchange ID** should reference a GTM Cookie Variable for `_kx` (e.g. `{{Cookie - _kx}}`)
- **Profile Properties** will update persistent fields on the Klaviyo profile
- **Event Volume Limit** is enforced with a `kmt-log` browser cookie, not a Klaviyo-side mechanism
  - For example, if set to `1`, the same event won't fire again from the same browser for 1 hour
  - The cookie is scoped to your top-level domain (e.g. `.corporatefinanceinstitute.com`), so the limit holds across subdomains
  - If you need to re-test an event during development, clear the `kmt-log` cookie in your browser's DevTools (Application → Cookies), or disable the "Limit how often this event tracks?" checkbox
- If both **email** and **_kx** are missing, the event will still be tracked anonymously (unless profile properties are used)

---

## 🧪 Testing

This template includes a test suite to verify behavior such as:
- Firing `identify` with email or `$exchange_id`
- Correctly formatting `value` and typed properties
- Preventing tracking if no event name is set

> **Note:** The cookie-based volume limiting introduced in this version (`getCookieValues`/`setCookie`) is not yet covered by the automated test suite above — it's been verified through manual/live testing (see the debug session notes) but would benefit from dedicated cookie-mocked test cases in a future update.

---

## 📄 License

MIT © 2025 - Ross Hopkins

---

## 📬 Future Plans

- **Array-type support for Profile Properties.** Event Properties already accept comma-separated arrays; Profile Properties currently only support text, number, and boolean. Mirroring array support across is a planned near-term addition.
- A server-side version (`gtm-server-klaviyo-metric-tracker`) is planned for use with GTM server containers and Klaviyo’s Event API.

---

## 🙌 Credit

Built and maintained by [PoshVilla] (Ross Hopkins) for the GTM and Klaviyo community.
