# GTM Client - Klaviyo Metric Tracker

## TLDR;

Google Tag Manager template for tracking Klaviyo events client-side using _learnq. Identify users and send custom metrics without writing code or using custom HTML tags.

## A little more info...

A Google Tag Manager tag template for sending custom events (metrics) to Klaviyo on the **client-side**, without writing any JavaScript.

This template lets you track custom user interactions—such as product views, quiz completions, signups, or any other meaningful activity—and pass event data to Klaviyo for segmentation, flow logic, or reporting. It also handles user identification via email or the Klaviyo cookie (`_kx`), and supports optional properties, event value fields, and profile updates.

---

## ✅ Features

- No need for custom HTML tags
- Identify users via email or `$exchange_id` (`_kx` cookie)
- Send event name, `value`, and unlimited custom event properties
- Optionally include **Profile Properties** (for updating user-level fields)
- Optionally enable **Event Volume Limit** (prevent duplicate tracking per time window)
- Supports typed properties: text, number, or array (comma-separated)
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
4. Add a trigger (e.g. form submission, page view)
5. Preview & publish

---

## 🛠 Field Notes

- **Value** is auto-converted to a number with 2 decimal places
- **Array-type** properties should be comma-separated (e.g. `Program,Certification,B2C`)
- **Exchange ID** should reference a GTM Cookie Variable for `_kx` (e.g. `{{Cookie - _kx}}`)
- **Profile Properties** will update persistent fields on the Klaviyo profile
- **Event Volume Limit** adds a time-bucketed `$event_id` to deduplicate repeated events
  - For example, if set to `1`, events will only track once per hour per profile
- If both **email** and **_kx** are missing, the event will still be tracked anonymously (unless profile properties are used)

---

## 🧪 Testing

This template includes a full test suite to verify behavior such as:
- Firing `identify` with email or `$exchange_id`
- Correctly formatting `value` and typed properties
- Applying `$event_id` deduplication window correctly
- Preventing tracking if no event name is set

---

## 📄 License

MIT © 2025 - Ross Hopkins

---

## 📬 Future Plans

A server-side version (`gtm-server-klaviyo-metric-tracker`) is planned for use with GTM server containers and Klaviyo’s Event API.

---

## 🙌 Credit

Built and maintained by [Ross] for the GTM and Klaviyo community.
