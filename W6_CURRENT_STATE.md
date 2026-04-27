# W6 WhatsApp Menu Bot — Current State (2026-04-26)

## ✅ Veikia
- Webhook gauna inbound iš Twilio Production Sender (+972533304556)
- WebhookRespond grąžina tuščią TwiML `<Response></Response>` (NEBĖRA "Accepted" žinutės klaidos)
- HTTP modulis siunčia `root_menu_he` template'ą per Twilio Messages API
- Bot atsako Hebrew root_menu (su mygtuku "בחר אפשרות")

## ❌ Trūksta
- Sesijų tracking Google Sheets (`WA_Sessions` lentelė)
- Kalbos detekcija iš inbound žinutės (he/ru/en/lt) — dabar visada Hebrew
- 15 routes pagal `current_step` ir button click
- Cal.com booking (3 lygiai: dates → times → confirm)
- Error handling
- Telegram alerts

## 🔑 Credentials & IDs

### Twilio
- Account SID: `<TWILIO_ACCOUNT_SID>` (žr. .env.local)
- API Key SID: `<TWILIO_API_KEY_SID>`
- API Key Secret: `<TWILIO_API_KEY_SECRET>`
- WhatsApp From: `whatsapp:+972533304556`
- API URL: `https://api.twilio.com/2010-04-01/Accounts/{AccountSid}/Messages.json`

### Make.com
- Scenario ID: `4866621`
- Webhook Hook ID: `2218244`
- Webhook URL: `https://hook.us2.make.com/3yw62ufgqlrkpqywe6tnhppw7lt6c4z9`
- Team ID: `1617748`
- Google Sheets connection ID: `8110896`
- Telegram connection ID: `8406260`
- Telegram chat ID: `1034122765`

### Google Sheets
- Spreadsheet ID: `1rTDUt_j4LHa9_rGFBpj_977YoQcM0jxMOp_NjMzWlPk`
- Sheet name: `WA_Sessions`

### Cal.com
- API Key: `<CALCOM_API_KEY>` (žr. .env.local)
- Event Type ID: `4880917`

## 📨 Content SIDs (visi 36 patvirtinti)

```
petruskevich_root_menu_he=HX5f9ab1f6ffbf4759bfdba29f141e9d71
petruskevich_root_menu_ru=HXfecd7a43aa9fe9b3bee4a847c991d823
petruskevich_root_menu_en=HXae47a283a2c1c0e47db5e1b926134817
petruskevich_root_menu_lt=HX7748c82210214509d14d5b6215692c42

petruskevich_website_l1_he=HX9c93738967cc510f713b1c00d87b762e
petruskevich_website_l1_ru=HX65c73cb06940dd909b47d0c4fac61944
petruskevich_website_l1_en=HX17cbe2b6e6188d81ae8d4d30b3bc858f
petruskevich_website_l1_lt=HX09b9170f744983e0419fddb2c2a2fe7c

petruskevich_website_l2_he=HX5094c1d46e00b822b1803908ebada1d5
petruskevich_website_l2_ru=HXf93254ab4ba511020ff0954921f84f5c
petruskevich_website_l2_en=HX898ea986ad91213bee565f61129d8e31
petruskevich_website_l2_lt=HX0f87f0fdb6a71eab758ca332aa404539

petruskevich_aibot_menu_he=HX803a1a016921b1c87a000a4f9bb23010
petruskevich_aibot_menu_ru=HXe6940a92b57e9e8a301116d7b7aa10fc
petruskevich_aibot_menu_en=HX0ae83e2c8a8fb33f239a0bbda025ae9b
petruskevich_aibot_menu_lt=HX0d111afb556194c3c5ab3a08f31abc99

petruskevich_aisec_menu_he=HXeeea3ad55c17dd2d2fac992a83ff4300
petruskevich_aisec_menu_ru=HXbde14e19b51d722391788f4ab39255f4
petruskevich_aisec_menu_en=HXe8c877c5968fa400cb89932b24ee645b
petruskevich_aisec_menu_lt=HX6739c1dbcb148faaaaab834c6223370b

petruskevich_cal_dates_he=HXc811eacba7dd13e7686de9c94ba0f385
petruskevich_cal_dates_ru=HXd462257b68225f947c9fe6e150e81d83
petruskevich_cal_dates_en=HX768f3470404f64824461d3677a18cb68
petruskevich_cal_dates_lt=HXcb2f7e5282cf7d04fd53be05ff71bb4a

petruskevich_cal_times_he=HXccadf981b37e50a6c17ac9b6431a8dff
petruskevich_cal_times_ru=HXbe6a0d826f29611880ab84288e08baaf
petruskevich_cal_times_en=HX19963c4b25f7c93779fc9e9569be77bf
petruskevich_cal_times_lt=HX111405f971d2d901a25d0495fe52d3a7

petruskevich_confirm_he=HX303036ae5fe36828030839c1d537bb55
petruskevich_confirm_ru=HXce455a32ca9b68d55082d58e7aa7270f
petruskevich_confirm_en=HX5ea0e30a14fefa730bc734f2e24d1578
petruskevich_confirm_lt=HXed4ef5dd2a79bc4a12298be446928b41

petruskevich_error_he=HX5d3daabfe24750c88dc36b00f6da9107
petruskevich_error_ru=HX254ccbbe31fa801750d5aa7de767a117
petruskevich_error_en=HX9c51695db9ee1994ef1113d8f2ffe532
petruskevich_error_lt=HXe338436704251b5c78ecc25232a6650e
```

## 🛠️ Make HTTP Modulio Schema (kuri VEIKIA su Twilio)

KRITINIS — naudok šiuos parametrus, NE `application/x-www-form-urlencoded` (nėra Make enum'e):

```json
{
  "module": "http:ActionSendData",
  "version": 3,
  "parameters": {"handleErrors": false, "useNewZLibDeCompress": true},
  "mapper": {
    "url": "https://api.twilio.com/2010-04-01/Accounts/<TWILIO_ACCOUNT_SID>/Messages.json",
    "method": "post",
    "headers": [],
    "qs": [],
    "bodyType": "raw",
    "contentType": "text/plain",
    "data": "From=whatsapp%3A%2B972533304556&To={{...}}&ContentSid={{...}}",
    "authUser": "{TWILIO_API_KEY_SID}",
    "authPass": "{TWILIO_API_KEY_SECRET}",
    "ca": "", "qs": [], "timeout": "30", "useMtls": false,
    "serializeUrl": false, "shareCookies": false,
    "parseResponse": true, "followRedirect": true,
    "useQuerystring": false, "followAllRedirects": false,
    "rejectUnauthorized": true, "gzip": true
  }
}
```

**Twilio API parsuoja form-urlencoded body net jei Content-Type: text/plain — tikrinta curl'u.**

## 📋 Originalus 15-Routes Planas

### Webhook payload fields (Twilio inbound)
- `Body` — text typed
- `From` — `whatsapp:+972XXXXXXXXX`
- `ProfileName` — kontakto vardas WhatsApp'e
- `ButtonText` / `ButtonPayload` — kai paspaudžiamas Quick Reply mygtukas
- `ListId` / `ListTitle` — kai pasirenkama iš List

### Variables to set after webhook
```javascript
phone_clean = replace(replace(replace(replace(1.From; "whatsapp:"; ""); "+"; ""); "-"; ""); " "; "")
user_input = ifempty(1.ListItemId; ifempty(1.ButtonPayload; 1.Body))
is_button_click = if(1.ListItemId; "true"; if(1.ButtonPayload; "true"; "false"))
language = ifempty(sheets.language; "he")  // default Hebrew
```

### Routes
1. **ROUTE_1: New session OR expired (>48h)**
   - `session_exists=false OR is_expired=true`
   - Detect language from raw_message (`labas`→lt, `привет`→ru, `hello`→en, default→he)
   - addRow to WA_Sessions: phone, lang, current_step="root", last_message_at=now
   - Send `HX_ROOT_MENU_{lang}`

2. **ROUTE_2: Root menu choice**
   - `current_step=root`
   - sub: `service_website` → updateRow step=website_l1, send `HX_WEBSITE_L1_{lang}`
   - sub: `service_aibot` → step=aibot, send `HX_AIBOT_MENU_{lang}`
   - sub: `service_aisec` → step=aisec, send `HX_AISEC_MENU_{lang}`
   - sub: `service_freechat` → forward to W5 webhook (open chat with AI)

3. **ROUTE_3: Website L1**
   - `current_step=website_l1`
   - Save subservice prefix (website_new or website_update)
   - step=website_l2, send `HX_WEBSITE_L2_{lang}`

4. **ROUTE_4: Website L2**
   - `current_step=website_l2`
   - Combine subservice (website_new_info, website_new_store, website_update_info, website_update_store)
   - If update → step=collect_website (ask for current site URL)
   - Else → step=collect_name (skip website link)
   - Send appropriate plain text question

5. **ROUTE_5: AI Bot subservice**
   - `current_step=aibot`
   - sub: aibot_reply, aibot_collect, aibot_store, aibot_chat
   - step=collect_website OR collect_name based on subservice

6. **ROUTE_6: AI Secretary subservice**
   - `current_step=aisec`
   - sub: aisec_phone, aisec_web, aisec_queue, aisec_sales

7. **ROUTE_7: Collect Website URL**
   - `current_step=collect_website`
   - Save Body as collected_website
   - step=collect_name, ask for name

8. **ROUTE_8: Collect Name**
   - `current_step=collect_name`
   - Save Body as collected_name
   - step=collect_email, ask for email

9. **ROUTE_9: Collect Email**
   - `current_step=collect_email`
   - Save Body as collected_email
   - step=cal_dates, fetch Cal.com slots, send `HX_CAL_DATES_{lang}`

10. **ROUTE_10: Cal Dates**
    - `current_step=cal_dates`
    - Save selected date
    - step=cal_times, fetch slots for that date, send `HX_CAL_TIMES_{lang}`

11. **ROUTE_11: Cal Times**
    - `current_step=cal_times`
    - Save selected time
    - step=confirm, send `HX_CONFIRM_{lang}` with summary

12. **ROUTE_12: Confirm Booking**
    - `current_step=confirm`
    - sub: yes → POST Cal.com /v2/bookings, step=done
    - sub: no → step=root, send root_menu

13. **ROUTE_13: Done state**
    - `current_step=done`
    - Reply "session ended, type anything to start over"
    - On any input → reset to step=root

14. **ROUTE_14: Free chat handoff**
    - When step=freechat
    - Forward all messages to W5 (AI chatbot) via webhook

15. **ROUTE_15: Error / Unknown**
    - Default fallback
    - Send `HX_ERROR_{lang}`
    - Telegram alert

## 🌍 Cal.com API
```
GET https://api.cal.com/v2/slots/available?eventTypeId=4880917&startTime={iso}&endTime={iso}
Authorization: Bearer cal_live_b48456f1bed309f86eaa514bb4ca27d0

POST https://api.cal.com/v2/bookings
Authorization: Bearer cal_live_b48456f1bed309f86eaa514bb4ca27d0
Body: {
  "eventTypeId": 4880917,
  "start": "2026-05-01T10:00:00Z",
  "responses": {"name": "...", "email": "..."},
  "timeZone": "Asia/Jerusalem",
  "language": "en"
}
```

## ⚠️ Žinomos problemos

1. **Make HTTP module schema validation per API:**
   - `bodyType: x_www_form_urlencoded` + array data → UnexpectedError
   - `contentType: application/x-www-form-urlencoded` → BundleValidationError (ne enum'e)
   - SPRENDIMAS: naudok `bodyType: raw` + `contentType: text/plain` (kaip dabar)

2. **`encodeURL` IML function neegzistuoja Make'e** — naudok manual:
   `replace(replace(text; ":"; "%3A"); "+"; "%2B")`

3. **`add()` su skaičiumi grąžina array error** — naudok `parseNumber(x) + 1`

4. **WebhookRespond su tuščiu body grąžina default "Accepted" stringą** — VISADA naudok valid TwiML `<Response></Response>` su `Content-Type: text/xml`
