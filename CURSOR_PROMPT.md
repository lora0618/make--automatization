# PROMPT CURSOR'UI — W6 WhatsApp Bot tęsimas

## Kontekstas

Šis projektas yra Petruskevich Web & AI Solutions WhatsApp bot'as. Bot'as turi 4 paslaugas su menu pasirinkimais, surenka kontaktinę informaciją ir registruoja klientus į susitikimą per Cal.com.

**Skaityk šiuos failus prieš pradedant:**
1. `W6_CURRENT_STATE.md` — kas dabar veikia, kas trūksta, visi credentials, Content SIDs, žinomos problemos
2. `W6_blueprint_current.json` — dabartinis Make scenarijus (3 moduliai, veikia)
3. `FLOW_SPEC.md` — TIKSLI flow specifikacija (TIK ką klientas paprašė, NIEKO daugiau)
4. `ANALYSIS_META_VS_TWILIO.md` — pasiūlymas Meta direct vs Twilio architektūra

## Užduotis

Užbaigti W6 Make scenarijų pagal `FLOW_SPEC.md`. NIEKO neišgalvoti — daryti TIK tai kas specifikacijoje.

## Esminiai apribojimai

1. **NEPRIDĖK funkcionalumo, kurio nėra FLOW_SPEC.md** (pvz. SMS notifications, email follow-ups — JEIGU TO NĖRA SPEC, NEDARYK)

2. **Make HTTP modulio schema** — naudok EXACT struktūrą iš `W6_blueprint_current.json`:
   - `bodyType: "raw"`
   - `contentType: "text/plain"`
   - `data` = manualiai URL-encoded string (ne array)
   - **NENAUDOK** `application/x-www-form-urlencoded` — NĖRA Make enum'e
   - **NENAUDOK** `bodyType: "x_www_form_urlencoded"` — duoda UnexpectedError

3. **WebhookRespond** — VISADA naudok `<Response></Response>` su `Content-Type: text/xml`. **NIEKADA tuščio body** — Make grąžina default "Accepted" plain text, kuris siunčiamas klientui.

4. **IML expressions:**
   - `encodeURL` neegzistuoja — naudok `replace(replace(text; ":"; "%3A"); "+"; "%2B")`
   - `add()` array error — naudok `parseNumber(x) + 1`

## Output

Atnaujink Make scenarijų ID `4866621` per Make MCP `scenarios_update` su pilnu flow pagal FLOW_SPEC.md. Po kiekvieno modify — testuok per `executions_list` ar nėra BundleValidationError.

Jei klaida — pataisyk modulį, NEMETK viso scenarijaus.

## GitHub

Sukurk naują GitHub projektą `make-automatizacija` ir push'ink visus failus iš `C:\Users\lpetr\wa-bot-final\`. Naudok `gh` CLI:

```bash
cd C:\Users\lpetr\wa-bot-final
gh repo create make-automatizacija --private --source=. --remote=origin
git add .
git commit -m "W6 WhatsApp bot — current state + flow spec"
git push -u origin main
```
