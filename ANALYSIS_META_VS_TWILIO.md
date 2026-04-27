# Architektūros pasiūlymas: Meta Direct vs Twilio ISV

> **Tikslas**: išsiaiškinti kuris kelias geriau Petruskevich WhatsApp bot'ui — tiesioginis Meta WhatsApp Business Cloud API, ar per Twilio (BSP — Business Solution Provider).
>
> **Šaltiniai**: tik faktai iš dabartinio chat'o ir patvirtinti dokumentai. NIEKO neišgalvota.

---

## Faktai apie dabartinę Lora būklę

### Twilio (DABAR aktyvu)
- ✅ Twilio ISV WhatsApp Sender registruotas: `+972533304556`, status ONLINE
- ✅ Susietas su Meta WABA `1850070728978992`
- ✅ Webhook → Make W6 (`https://hook.us2.make.com/3yw62ufgqlrkpqywe6tnhppw7lt6c4z9`)
- ✅ 36 Twilio Content Templates patvirtinti (root_menu, website_l1, etc.)
- ✅ Twilio API Key: `<TWILIO_API_KEY_SID>` su Secret (saugomi .env.local)
- ✅ Bot atsako klientams root_menu_he template'u

### Meta (TURI prieigą)
- ✅ Meta Business Manager + WABA `1850070728978992`
- ✅ App "Petruskevich" (App ID: `1525738885757145`) — Live mode
- ✅ App "Conversations API Application" (App ID: `1035375514376219`) — Live
- ⚠️ Petruskevich app webhook callback URL: TUŠČIAS (ne sukonfigūruotas tiesioginiam Cloud API)

---

## A) Meta Direct (Cloud API)

### Privalumai
1. **Pigiau** — Meta Cloud API ima ~$0.005-0.05/conversation (priklauso nuo šalies). Twilio papildomai +$0.005-0.01 markup.
2. **Mažesnis latency** — vienas mažiau hop'as (nėra Twilio BSP layer'o)
3. **Pilnas API access** — visi WhatsApp Business features tiesiogiai (Flows, Catalog, Payments etc.)
4. **Nepriklausoma nuo Twilio infrastruktūros** — jei Twilio sutrinka, bot'as veikia
5. **Native Meta tooling** — WhatsApp Manager UI, Insights, Quality Rating analytics tiesiai Meta pusėje

### Trūkumai
1. **Reikia naują Content Template tipo schemos** — Meta naudoja `template` su `name + language + components`. **Twilio ContentSids (HX...) NEVEIKIA su Meta direct API**.
2. **Reikia perregistruoti VISUS 36 templates** Meta WhatsApp Manager — kiekvienas turės naują `name`+`language` tuple. Twilio'iaus patvirtintos ContentSids prarandama.
3. **Webhook iššūkis** — Meta webhook reikalauja HTTPS endpoint su:
   - GET `?hub.challenge=` verification
   - X-Hub-Signature-256 SHA-256 HMAC validation
4. **Phone Number Migration** — `+972533304556` šiuo metu valdomas TWILIO BSP. Norint perkelti į Cloud API tiesiogiai, reikia "Migrate to Cloud API" procedūros Meta Business Manager (gali užimti 1-3 dienas, ir bot'as nebeveiks tuo metu).
5. **Token rotation** — Meta tokens 60 dienų galiojimo. Reikia automatinio refresh proceso (`WA Token Auto-Renewal` jau yra Make'e bet inactive).

### Kainos pavyzdys (1000 žinučių/mėn į Izraelį)
- Meta Cloud API: ~$10-15/mėn (service conversations)
- Twilio: ~$15-25/mėn (Meta cost + Twilio markup)

---

## B) Twilio (DABAR — current setup)

### Privalumai
1. **JAU VEIKIA** — sender registered, templates approved, API tested
2. **Lengvesnis Make integration** — Twilio Messages API standartinis HTTP POST su Basic Auth
3. **BSP teikia palaikymą** — jei Meta blokuoja, Twilio pagalbos lygis su Meta'a
4. **Content Templates UI** Twilio Console — pažįstama Lora'ai
5. **TwiML auto-responses** — galima atsakyti `<Response><Message>...</Message></Response>` be API call'o
6. **Studio Flows** prieinama (jei norėtų be-code flow)

### Trūkumai
1. **Brangiau** ~$5-10/mėn papildomai (Twilio markup)
2. **Papildomas hop** — latency padidėja ~100-300ms
3. **ISV setup quirks** — kai kuriuose endpoint'uose reikia ISV credentials
4. **"Accepted" bug** patyrėme — Make webhook default response triggered Twilio auto-send. **JAU IŠTAISYTA** su `<Response></Response>` TwiML.

---

## C) HIBRIDINIS (rekomendacija)

**Pasiūlymas**: TĘSTI su Twilio dabar (Maximize already-done work), bet ATEITY apsvarstyti migraciją į Meta direct kai bot'as bus stabiliai veikiantis 1-2 mėnesius.

### Pagrindas:
- **Dabartinis prioritetas** = užbaigti veikiantį bot'ą su 4 paslaugomis. Twilio jau setup'intas, šablonai patvirtinti, API testas pavyko. Migracija dabar = 1-2 savaitės papildomo darbo.
- **Po MVP launch** — kai matysime realų traffic ir kainos analizę, galima migruoti į Meta direct. Migracija užims 2-3 dienas su pertraukimu.

### Konkrečiai dabar daryti:
1. Užbaigti W6 Make scenarijų su Twilio
2. Stebėti bot'o veikimą 2-4 savaites
3. Skaičiuoti Twilio kaštus
4. **JEI** Twilio kaštai > $50/mėn ARBA latency problemos → migruoti į Meta direct su papildoma sprintų savaite

---

## Specifika 4-jam scenarijui (laisvas pokalbis su AI)

Lora'os schema:
```
Klientas → WhatsApp → Meta API (token) ARBA Twilio
              ↓
           Make.com webhook
              ↓
         OpenAI / Claude API
              ↓
         Make.com → Meta API → WhatsApp → Klientas
```

### Su Twilio (rekomenduojama dabar):
1. Twilio webhook → Make
2. Jei `current_step = freechat` → Make HTTP modulis kviečia OpenAI/Anthropic API
3. AI atsakymas → Make HTTP modulis siunčia atgal per Twilio Messages API kaip plain text
4. **Latency: ~1-3 sekundės** (Twilio + Make + OpenAI/Anthropic + Twilio)

### Su Meta direct (ateity):
1. Meta webhook → Make
2. AI logic Make'e
3. Meta Cloud API atgal
4. **Latency: ~0.7-2 sekundės** (~30% greičiau)

---

## Veiksmų sąrašas

### Trumpalaikiai (ŠIAME chat'e — Cursor)
1. Užbaigti Twilio-based W6 pagal `FLOW_SPEC.md`
2. Implementuoti visus 4 šakas (Website, AI Bot, AI Sec, Free Chat)
3. Sheets `whatsapp` integration
4. Telegram alerts
5. Test end-to-end

### Vidutinio laiko (1-2 mėn)
1. Stebėti realų vartotojų traffic
2. A/B testas Twilio vs Meta direct (galima paleisti Meta direct su antru numeriu lyginant)
3. Skaičiuoti tikras kainas

### Ilgalaikis (jei pasiteisina migracija)
1. Migrate `+972533304556` į Meta Cloud API
2. Perkurti 36 templates Meta'oje
3. Perkonfigūruoti Make scenarijus naudoti Meta endpoints
4. Testuoti viską
5. Switch over

---

## Galutinė rekomendacija

**TĘSK SU TWILIO DABAR.** Nepatariu migruoti į Meta direct kol nebus veikiančio MVP. Migracija = 1-2 savaitės papildomo darbo + rizika prarasti šablonų patvirtinimą.

**KAS DARYTI ŠIAME ETAPE:**
- Užbaigti W6 visus 15 routes pagal FLOW_SPEC.md
- Implementuoti laisvo pokalbio AI (Twilio + OpenAI per Make)
- Užbaigti Sheets + Telegram integracijas
- Testuoti su realiu klientu

Migracija į Meta direct = Phase 2 (kai bot'as veiks ir bus realių vartotojų).
