# W6 WhatsApp Bot — TIKSLI Flow Specifikacija

> **TAISYKLĖ**: Daryk TIK tai kas čia parašyta. Nieko neišgalvok. Nieko nepridėk.

## Bendri principai

- **Kalba**: aptikta iš pirmosios žinutės teksto. Default Hebrew (`he`) jeigu neaišku.
  - `labas`, `sveiki`, `Ar` → Lietuvių (`lt`)
  - `привет`, `здравствуйте` → Russian (`ru`)
  - `hi`, `hello`, `good` → English (`en`)
  - viską kita → Hebrew (`he`)
- **Visa informacija** įrašoma į Google Sheets:
  - Spreadsheet: `1rTDUt_j4LHa9_rGFBpj_977YoQcM0jxMOp_NjMzWlPk`
  - Sheet: `whatsapp` (pagal vartotojo specifikaciją)
- **Naujas lead'as** → siųsti pranešimą į Telegram bot'ą (chat ID `1034122765`)
- **Susitikimas**: Cal.com link `https://cal.com/petruskevich/30min`

## Pradžia (visi vartotojai)

Kai klientas rašo bot'ui pirmą kartą (arba po 48h tylos):
1. Aptikti kalbą iš pirmos žinutės
2. Įrašyti naują eilutę į Google Sheets `whatsapp`
3. Atsiųsti **root_menu_{lang}** template'ą su 4 mygtukais:
   - **1. Website** (אתרי אינטרנט / Сайты / Websites / Svetainės)
   - **2. AI Bot** (AI Bot)
   - **3. AI Sekretorė** (AI Secretary)
   - **4. Laisvas pokalbis** (Free chat)
4. Nustatyti `current_step = root` Sheets'e

## Flow šaka 1: WEBSITE

### Kai paspaudžia "1 Website":
- `current_step = website_l1`
- Atsiųsti **website_l1_{lang}** template'ą:
  - **1.1** Naujas website (חדש)
  - **1.2** Pataisyti egzistuojantį (עריכה)

### Kai paspaudžia "1.1 Naujas":
- `subservice = website_new`
- `current_step = website_l2_new`
- Atsiųsti **website_l2_{lang}** template'ą (su `subservice=new`):
  - **1.1.1** Informacinė svetainė
  - **1.1.2** El. parduotuvė

### Kai paspaudžia "1.2 Pataisyti":
- `subservice = website_update`
- `current_step = website_l2_update`
- Atsiųsti **website_l2_{lang}** template'ą (su `subservice=update`):
  - **1.2.1** Informacinė svetainė
  - **1.2.2** El. parduotuvė

### Kai paspaudžia 1.1.1 / 1.1.2 / 1.2.1 / 1.2.2:
- Įrašyti `subservice` (website_new_info, website_new_store, website_update_info, website_update_store)
- `current_step = collect_name`
- Klausti: **"Koks tavo vardas?"** (atitinkama kalba)

### Lead capture sequence (visi 4 variantai):
1. `current_step = collect_name` → klausia **vardo**
2. `current_step = collect_website` → klausia **saito linko**
3. `current_step = collect_email` → klausia **el. pašto**
4. `current_step = collect_phone` → klausia **telefono numerio** (default = `From` numeris)
5. `current_step = booking` → atsiųsti `https://cal.com/petruskevich/30min` ir prašyti pasirinkti laiką
6. Kai klientas patvirtina → `current_step = done`, įrašyti į Sheets, siųsti Telegram alert

## Flow šaka 2: AI BOT

### Kai paspaudžia "2 AI Bot":
- `current_step = aibot`
- Atsiųsti **aibot_menu_{lang}** template'ą:
  - **2.1** AI bot atsakinėti į žinutes (aibot_reply)
  - **2.2** AI bot rinkti duomenis (aibot_collect)
  - **2.3** AI bot suristi su el. parduotuve (aibot_store)
  - **2.4** AI botas laisvam bendravimui (aibot_chat)

### Kai paspaudžia 2.1 / 2.2 / 2.3 / 2.4:
- Įrašyti `subservice`
- `current_step = collect_name`
- Klausti **vardo**

### Lead capture sequence:
1. **vardas**
2. **saito linkas** — TIK JEI `subservice=aibot_store` (2.3 atveju). Kitiems variantams praleisti.
3. **el. paštas**
4. **telefono numeris**
5. Cal.com booking link
6. Sheets save + Telegram alert

## Flow šaka 3: AI SEKRETORĖ

### Kai paspaudžia "3 AI Sekretorė":
- `current_step = aisec`
- Atsiųsti **aisec_menu_{lang}** template'ą:
  - **3.1** Atsakinėti į skambučius telefonu (aisec_phone)
  - **3.2** Atsakinėti per tinklapį (aisec_web)
  - **3.3** Registruoti eiles (aisec_queue)
  - **3.4** Pardavimai iš el. parduotuvės (aisec_sales)

### Kai paspaudžia 3.1 / 3.2 / 3.3 / 3.4:
- Įrašyti `subservice`
- `current_step = collect_name`
- Klausti **vardo**

### Lead capture sequence:
1. **vardas**
2. **saito linkas** — TIK JEI `subservice=aisec_web` ARBA `subservice=aisec_sales` (3.2 arba 3.4 atveju). Kitiems variantams praleisti.
3. **el. paštas**
4. **telefono numeris**
5. Cal.com booking link
6. Sheets save + Telegram alert

## Flow šaka 4: LAISVAS POKALBIS SU AI

### Kai paspaudžia "4 Laisvas pokalbis":
- `current_step = freechat`
- Pradedi laisvą AI pokalbį

### Architektūra:
```
Klientas → WhatsApp → Twilio Webhook → Make W6
                                          ↓
                            (jei step=freechat)
                                          ↓
                        OpenAI / Claude API call
                                          ↓
                        Make → Twilio API → WhatsApp → Klientas
```

### Detali logika:
1. Make modulis tikrina ar `current_step = freechat`
2. Jei taip — žinutės tekstas (`Body`) siunčiamas į **OpenAI GPT-4** arba **Anthropic Claude** API
3. **System prompt**: "Tu esi Petruskevich Web & AI Solutions paslaugų agentas. Naudok verslo žinių bazę: [FAQ + paslaugos + kainos]. Atsakyk klientiškai. Atsako kalba: {lang}"
4. AI atsakymas siunčiamas atgal per Twilio Messages API kaip plain text žinutė
5. Po kelių žinučių (3-5 atsakymų) ARBA jei klientas pasako "noriu kalbėti" / "rezervacija" / "noriu pirkti" — pereiti į lead capture:
   - **vardas**
   - **saito linkas**
   - **el. paštas**
   - **telefono numeris**
   - Cal.com booking
   - Sheets save + Telegram alert

### TODO: laukti vartotojo papildomos info kaip nustatyti AI verslo žinių bazę (FAQ, kainos, paslaugos). KOL KAS placeholder.

## Google Sheets struktūra (whatsapp sheet)

| Stulpelis | Pavadinimas | Aprašymas |
|-----------|-------------|-----------|
| A | phone | Be `+` ir `whatsapp:` prefix |
| B | profile_name | Iš Twilio `ProfileName` |
| C | language | he / ru / en / lt |
| D | current_step | new, root, website_l1, ..., done |
| E | service_chosen | website / aibot / aisec / freechat |
| F | subservice | pvz. website_new_info |
| G | collected_name | |
| H | collected_website | |
| I | collected_email | |
| J | collected_phone | |
| K | first_message_at | timestamp |
| L | last_message_at | timestamp |
| M | booking_url_sent | true/false |
| N | booking_confirmed | true/false |
| O | freechat_history | (jei freechat) JSON array of messages |

## Telegram alert format

Kai klientas pateikia visus 4 lauką (vardas, saitas, email, telefonas) IR susitikimas užregistruotas:

```
🎯 NAUJAS LEAD!

📞 Tel: {{phone}}
👤 Vardas: {{collected_name}}
🌐 Saitas: {{collected_website}}
📧 Email: {{collected_email}}

🛠 Paslauga: {{service_chosen}} → {{subservice}}
🌍 Kalba: {{language}}
📅 Susitikimas: {{cal_booking_link}}

🕒 {{now}}
```

Siųsti į Telegram chat ID `1034122765` per connection `8406260`.

## Cal.com integracija

- **Public booking URL** (siunčiamas klientui): `https://cal.com/petruskevich/30min`
- **API key** (jei reikia automatinės rezervacijos): `<CALCOM_API_KEY>` (žr. .env.local)
- **Event Type ID**: `4880917`

Pradžioje — TIESIOG SIŲSTI URL kaip plain text žinutę. Klientas pats spustelės ir pasirinks laiką.

## Content SID lentelė (visi 36)

```
root_menu_{lang}     → 4 SID (he/ru/en/lt)
website_l1_{lang}    → 4 SID
website_l2_{lang}    → 4 SID
aibot_menu_{lang}    → 4 SID
aisec_menu_{lang}    → 4 SID
cal_dates_{lang}     → 4 SID (NEUŽGALI naudoti šiame fully MVP — Cal.com viduje)
cal_times_{lang}     → 4 SID (kaip aukščiau)
confirm_{lang}       → 4 SID (po booking confirmation)
error_{lang}         → 4 SID (klaidos)
```

Pilnas sąrašas: `C:\Users\lpetr\wa-bot-final\SHARED_STATE\content_sids.txt`

## TODO klausimai vartotojui

Šie punktai NEAIŠKŪS iš spec — paklausk vartotojo prieš darant:

1. **Šakos 4 (laisvas pokalbis) AI prompt** — koks turi būti system prompt? Kokia žinių bazė (FAQ, paslaugos, kainos)? KOL KAS PLACEHOLDER.
2. **Šakos 4 (laisvas pokalbis) trigger į lead capture** — po kiek žinučių pereiti į vardo/saito/email/tel klausimą? KOL KAS = po 5 žinučių ARBA jei klientas pasako "rezervacija"/"susitikimas"/"noriu kalbėti".
3. **Sheet `whatsapp`** — ar lentelė jau egzistuoja? Ar reikia sukurti su antraštėmis?

## Žinomos Make problemos (jau ištaisytos)

- ✅ "Accepted" reply: WebhookRespond su tuščiu body grąžina default "Accepted" plain text → naudok `<Response></Response>` su `Content-Type: text/xml`
- ✅ HTTP modulio Twilio API call: naudok `bodyType: "raw"` + `contentType: "text/plain"` + manualiai URL-encoded `data` string

## Tolesni žingsniai darbo eilėje

1. ✅ Root menu (jau veikia)
2. Pridėti Sheets `searchRows` lookup
3. Pridėti `SetVariables` modulį (phone_clean, user_input, current_step, language, is_button_click)
4. Pridėti `BasicRouter` su filtrais
5. Implementuoti ROUTE_1 (new session) — addRow + send root_menu_{lang}
6. Implementuoti ROUTE_2 (root → website/aibot/aisec/freechat)
7. Implementuoti ROUTE_3-6 (website_l1, website_l2, aibot, aisec)
8. Implementuoti collect_name → collect_website → collect_email → collect_phone sekvencijas
9. Implementuoti booking step (siusti Cal.com URL)
10. Implementuoti freechat (OpenAI/Claude integration)
11. Implementuoti Telegram alert
12. Implementuoti error fallback
