# Aynivra · Real Estate landing page (V3)

The page goes live at **https://aynivra.ai/realestate/**.

## What's in this folder

```
realestate/
  index.html             the whole page (HTML, CSS and JS in one file)
  assets/                images, video, logo, favicon, OG image
  assets/audio/          add stage-1.mp3 … stage-5.mp3 here (see below)
```

- Upload the `realestate` folder as it is.
- There is no build step. The page only loads Google Fonts from outside the site.
- The page is set to `noindex`. Remove that meta tag in `<head>` when we want it in Google.

## 1. Footer (web host)

- Find the `<footer>` near the bottom of `index.html`. It is marked **SITE FOOTER (placeholder)**.
- Replace the whole `<footer>` with the aynivra.ai site footer.
- Keep the thin disclaimer strip at the top of the page.
- Keep the note under the dashboard image.

## 2. Webhook (builder)

At the top of the last `<script>` in `index.html`, set:

```js
var WEBHOOK_URL = "https://<your-n8n>/webhook/realestate-demo";
```

- Until this is set, the form still works on the page, but nothing is sent. The payload is logged to the browser console instead.
- The n8n webhook must allow CORS from `https://aynivra.ai`.

Every request is a `POST` with a JSON body. There are two events.

### `demo_setup`: sent when the visitor clicks "Set up my demo agent"

| Field | Example | Notes |
|---|---|---|
| `event` | `demo_setup` | |
| `submission_id` | `5f0c…` | Same ID on all later triggers from this visit |
| `page` | `realestate` | |
| `salutation` | `Mr` | |
| `name` | `Govind Jaidhara` | required |
| `mobile` | `+919876543210` | required, validated as an Indian mobile number |
| `email` | `govind@…` | optional |
| `company` | `Reliable Builders` | optional |
| `role` | `Founder / Director` | Founder / Director · CMO / Head of sales · CFO / Head of accounts · Channel partner · Other |
| `project_name` | `Balaji Kripa` | required |
| `developer_group` | `Reliable Builders` | Falls back to `company` if left blank |
| `configurations` | `2 BHK (650 sq ft carpet), 3 BHK (890 sq ft carpet)` | Text version, ready for the agent prompt |
| `configurations_list` | `[{"type":"2 BHK","carpet_sqft":"650"}]` | Array version |
| `starting_price` | `78 lakh onwards` | optional, free text |
| `amenities` | `Swimming pool, Gym, Clubhouse` | Up to 5. Text version |
| `amenities_list` | `["Swimming pool","Gym"]` | Array version |
| `locality` | `5 min to Kharghar metro…` | Free text |
| `website` | `https://…` | optional |
| `language` | `Hindi` | Only Hindi at launch |
| `consent` | `true` | The form can't be sent without it |
| `source` | `qr` | From `?src=` or `?utm_source=`; `direct` if neither |
| `utm_campaign` | | |
| `page_url` | | |
| `submitted_at` | ISO time | |

**n8n should:**
- create or update the lead in the CRM, matching on `mobile`;
- add a row to the dashboard sheet;
- return `200`.

If the webhook returns an error, the visitor sees an error message and a WhatsApp link.

### `demo_trigger`: sent when the visitor clicks a use-case button

This is the same body as `demo_setup`, plus three fields:

| Field | Values |
|---|---|
| `event` | `demo_trigger` |
| `use_case` | `followup_call` · `emi_reminder` · `web_enquiry` |
| `triggered_at` | ISO time |

**n8n should:**
- `followup_call`: start the outbound follow-up call to `mobile`.
- `emi_reminder`: start the outbound EMI reminder call to `mobile`.
- `web_enquiry`: only log it. The browser starts this call itself (section 3).
- For both outbound calls, pass `project_name`, `developer_group`, `configurations`, `starting_price`, `amenities`, `locality` and `name` to the agent as call variables.
- Log every trigger against the lead in the CRM and in the sheet, so we can see which journeys each prospect tried.

## 3. "Enquire now" browser call (builder)

- Define `window.AynivraStartWebCall` before the page script runs, or replace the `null` in the settings block.
- It receives the full payload, so pass the project fields to the assistant as variables.
- It must return a Promise that resolves once the call has connected.
- Until it is wired, the button says browser calls aren't on yet and points the visitor to "Call me now".
- Test on mobile Chrome and Safari. The browser asks for microphone permission, and Safari is stricter about it.

## 4. Call recordings on the buyer journey

- Add `assets/audio/stage-1.mp3` … `stage-5.mp3`.
- A player whose file is missing shows "Recording coming soon" and is disabled. You don't need to change any code.

## 5. Before go-live checklist

- [ ] `WEBHOOK_URL` is set, and a test lead reaches both the CRM and the dashboard sheet.
- [ ] The follow-up and EMI calls ring a test phone within about a minute, speaking Hindi and using the project details.
- [ ] The web call works on Android Chrome and iPhone Safari.
- [ ] The site footer has replaced the placeholder.
- [ ] The QR code points to `https://aynivra.ai/realestate/?src=qr`.
