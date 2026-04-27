# W6 WhatsApp Bot — Darbo Istorija

> Kas buvo padaryta, kokios problemos rastos, kaip sutaisytos.
>
> Data: 2026-04-26

## Tikslas

Sukurti WhatsApp bot'ą Petruskevich Web & AI Solutions klientams su 4 paslaugomis (Website, AI Bot, AI Sekretorė, Laisvas pokalbis su AI), surenkančiu kontaktus ir registruojančiu į Cal.com susitikimą.

## Pradinė architektūra

```
Klientas (WhatsApp app)
    ↓
Twilio WhatsApp Sender (+972533304556)
    ↓ webhook
Make.com Scenario W6
    ↓
Twilio Messages API (atgal kliente)
    +
Google Sheets (sesijų log)
    +
Cal.com booking link
    +
Telegram alerts
```

## Etapas 1: Google Sheets ✅ PAVYKO
- Sukurta `WA_Sessions` lentelė su pilnais formatais ir validacijomis
- Spreadsheet ID: `1rTDUt_j4LHa9_rGFBpj_977YoQcM0jxMOp_NjMzWlPk`

## Etapas 2: Cal.com ✅ PAVYKO
- API key sukurtas: `cal_live_b48456f1bed309f86eaa514bb4ca27d0`
- Event Type ID: `4880917`
- API testai praeina

## Etapas 3: Twilio Content Templates ✅ PAVYKO
- 36 templates patvirtinti (root_menu, website_l1/l2, aibot_menu, aisec_menu, cal_dates/times, confirm, error)
- Visi 4 kalbos (he/ru/en/lt)
- SIDs surinkti į `SHARED_STATE/content_sids.txt`

## Etapas 4: Make scenarijus W6

### Bandymas 1-11: BundleValidationError loop ❌
- Bandyta pridėti pilną 35-modulių blueprint per `scenarios_update` API
- Kiekvienas bandymas grąžino `BundleValidationError: Validation failed for 1 parameter(s).`
- Make API nedraugiškas validuojant complex schemų (HTTP modulio bodyType variantai, contentType enum'ai)

### Esmine pamoka:
- Make HTTP module v3 contentType `application/x-www-form-urlencoded` **NĖRA** enum'e — duoda BundleValidationError
- Galimi enum'ai: `text/plain`, `application/json`, `application/xml`, `text/xml`, `text/html`, `custom`

### Bandymas 12-15: "Accepted" mįslė ❌
- Bot'as visada siuntė klientui žinutę su tekstu "Accepted"
- Pradžioje manyta kad tai Make WebhookRespond bug'as
- Bandyta keisti body į `{{emptystring}}`, ` ` (space), `<Response/>` — visi neveikė
- Lora suerzino: "tu viska tik salini bet niekas nesitaiso"

### Šaltinio paieška: ❌→✅
1. Patikrintas Twilio Sender → webhook URL = Make W6 ✅ (švari config)
2. Patikrintas Twilio Studio → 0 flows ✅
3. Patikrintas Twilio Functions → 0 services ✅
4. Patikrintos Twilio Conversations → 0 webhooks ✅
5. **Atrastas Hetzner serveris** (api.petruskevich.com) su PM2 procesu `whatsapp-bot`
   - Health endpoint: `https://api.petruskevich.com/health` → `{"status":"ok","service":"petruskevich-whatsapp-bot"}`
   - PM2 procesas online 3 dienas
   - **Spėta** kad jis siunčia "Accepted"
6. PM2 stop'intas → "Accepted" VIS DAR ATĖJO
7. Stop'inta visų 5 PM2 procesų → "Accepted" vis dar atėjo
8. **Tikra priežastis rasta**: tiesioginis curl į Make webhook URL grąžino body `Accepted` (8 ženklai) ir Content-Type: text/plain

### Sprendimas: ✅
Make WebhookRespond su `body: ""` (tuščiu) grąžino default fallback string `"Accepted"`. Twilio ISV WhatsApp BSP layer interpretavo plain text body kaip žinutę → konvertavo į Outgoing API call su `Body=Accepted`.

**Fix**: WebhookRespond `body: "<Response></Response>"` su `Content-Type: text/xml` header'iu. Tai validus tuščias TwiML, kurį Twilio supranta kaip "atsakyti niekuo".

### Etapas 5: HTTP modulis siunčia template ✅ PAVYKO
- Bandyta `bodyType: x_www_form_urlencoded` + array data → UnexpectedError
- Bandyta `bodyType: raw` + `contentType: custom` → BundleValidationError
- **Veikiantis**: `bodyType: raw` + `contentType: text/plain` + manualiai URL-encoded `data` string
- Twilio API parsuoja form-urlencoded body net jei Content-Type: text/plain (testuota curl'u)
- Lora gavo Hebrew `root_menu_he` template'ą su mygtuku "בחר אפשרות" ✅

## Dabartinė padėtis

### Veikiantis W6 (3 moduliai):
1. CustomWebHook (gauna inbound iš Twilio)
2. WebhookRespond (`<Response></Response>`, status 200, Content-Type: text/xml)
3. HTTP ActionSendData (siunčia root_menu_he template per Twilio API)

### Trūksta:
- Sesijų tracking (Google Sheets)
- Kalbos detekcija (dabar visada Hebrew)
- 15 routes pagal current_step + button click
- Cal.com booking flow
- Free chat AI integration
- Telegram alerts

## Žinomos problemos / sprendimai

| Problema | Priežastis | Sprendimas |
|----------|-----------|------------|
| Bot grąžina "Accepted" žinutę | Make WebhookRespond default fallback string | `body: "<Response></Response>"` + `Content-Type: text/xml` |
| BundleValidationError per scenarios_update | `application/x-www-form-urlencoded` ne Make enum'e | `bodyType: raw` + `contentType: text/plain` + URL-encoded data string |
| `encodeURL` IML function not found | Make IML neturi šios funkcijos | `replace(replace(text; ":"; "%3A"); "+"; "%2B")` |
| `add()` array error | `add()` skaičiui = klaida | `parseNumber(x) + 1` |
| UnexpectedError HTTP module | Schema nesutaria su Make backend | Vengti `x_www_form_urlencoded` bodyType |

## Resursai

- **Make scenarijus**: `https://us2.make.com/4866621`
- **Twilio Console**: `https://console.twilio.com/`
- **Hetzner serveris**: `77.42.73.200` (PM2 procesas `whatsapp-bot` pažymėta kaip ne susijęs su problema)
- **Meta Business**: `https://business.facebook.com/wa/manage/`
