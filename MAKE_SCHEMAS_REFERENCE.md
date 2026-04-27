# Make Module Schemas — TIKSLŪS formatai

> Šie schemų formatai paimti iš WORKING WA Sender (id 4673570) scenarijaus.
> Cursor agent ankstesnis blueprint (PR #1) nepraėjo Make import dėl klaidingos sheet/filter sintaksės — Make rodė "Invalid blueprint".
> Šis dokumentas turi tikslias schemas — naudok ŠIAS, ne savo supaprastintas.

---

## google-sheets:searchRows

```json
{
  "id": 10,
  "module": "google-sheets:searchRows",
  "version": 2,
  "parameters": {
    "__IMTCONN__": 8110896
  },
  "mapper": {
    "from": "drive",
    "mode": "fromAll",
    "limit": "1",
    "filter": [
      [
        {
          "a": "A",
          "b": "{{replace(replace(replace(replace(1.From; \"whatsapp:\"; \"\"); \"+\"; \"\"); \"-\"; \"\"); \" \"; \"\")}}",
          "o": "text:equal"
        }
      ]
    ],
    "sheetId": "whatsapp",
    "sortOrder": "asc",
    "spreadsheetId": "1rTDUt_j4LHa9_rGFBpj_977YoQcM0jxMOp_NjMzWlPk",
    "tableFirstRow": "A1:Z1",
    "includesHeaders": true,
    "valueRenderOption": "FORMATTED_VALUE",
    "dateTimeRenderOption": "FORMATTED_STRING"
  },
  "metadata": {
    "designer": {"x": 600, "y": 0}
  }
}
```

KEY POINTS:
- `filter` is ARRAY of ARRAYS of condition objects: `[[{a, b, o}]]`
- `a` = column letter (A, B, C...) NOT field name
- `o` = operator (text:equal, text:notequal, text:contain, exist, notexist)
- `b` = value to match
- Field access uses `__ROW_NUMBER__` and column letters: `10.A`, `10.B`, `10.D` etc.
- Need: `tableFirstRow`, `includesHeaders`, `valueRenderOption`, `dateTimeRenderOption`

---

## google-sheets:addRow

```json
{
  "id": 101,
  "module": "google-sheets:addRow",
  "version": 2,
  "parameters": {
    "__IMTCONN__": 8110896
  },
  "mapper": {
    "from": "drive",
    "mode": "fromAll",
    "values": {
      "0": "{{11.phone_clean}}",
      "1": "{{1.ProfileName}}",
      "2": "{{11.language}}",
      "3": "root",
      "4": "",
      "5": "",
      "6": "",
      "7": "",
      "8": "",
      "9": "",
      "10": "{{11.now_iso}}",
      "11": "{{11.now_iso}}",
      "12": "false",
      "13": "false",
      "14": "[]"
    },
    "sheetId": "whatsapp",
    "spreadsheetId": "1rTDUt_j4LHa9_rGFBpj_977YoQcM0jxMOp_NjMzWlPk",
    "includesHeaders": true,
    "tableFirstRow": "A1:O1",
    "insertUnformatted": false,
    "valueInputOption": "USER_ENTERED",
    "insertDataOption": "INSERT_ROWS"
  },
  "metadata": {
    "designer": {"x": 1500, "y": -600}
  }
}
```

KEY POINTS:
- `values` keys are **column INDICES as strings** ("0", "1", "2"...) NOT names!
- `tableFirstRow` defines column range (A1:O1 means 15 columns)
- Need: `valueInputOption`, `insertDataOption`, `insertUnformatted`

---

## google-sheets:updateRow

```json
{
  "id": 200,
  "module": "google-sheets:updateRow",
  "version": 2,
  "parameters": {
    "__IMTCONN__": 8110896
  },
  "mapper": {
    "from": "drive",
    "mode": "select",
    "values": {
      "3": "website_l1",
      "4": "website",
      "11": "{{11.now_iso}}"
    },
    "sheetId": "whatsapp",
    "rowNumber": "{{10.__ROW_NUMBER__}}",
    "spreadsheetId": "1rTDUt_j4LHa9_rGFBpj_977YoQcM0jxMOp_NjMzWlPk",
    "includesHeaders": true,
    "useColumnHeaders": false,
    "valueInputOption": "USER_ENTERED"
  },
  "metadata": {
    "designer": {"x": 1500, "y": -360}
  }
}
```

KEY POINTS:
- `mode: "select"` (not "fromAll")
- `values` indexed by column number string
- `rowNumber` references searchRows output `__ROW_NUMBER__`
- `useColumnHeaders: false`

---

## builtin:BasicRouter

```json
{
  "id": 20,
  "module": "builtin:BasicRouter",
  "version": 1,
  "mapper": null,
  "parameters": {},
  "metadata": {
    "designer": {"x": 1200, "y": 0}
  },
  "routes": [
    {
      "filter": {
        "name": "ROUTE_1 new session",
        "conditions": [
          [
            {
              "a": "{{10.__ROW_NUMBER__}}",
              "o": "notexist"
            }
          ]
        ]
      },
      "flow": [
        // ... modules in this route
      ]
    }
  ]
}
```

KEY POINTS:
- `filter.conditions` is ARRAY of ARRAYS (outer = OR logic, inner = AND)
- Each condition: `{a, o, b?}` where `b` only for binary operators
- Operators: `text:equal`, `text:notequal`, `exist`, `notexist`, `text:contain`
- Inner `flow` contains module objects

---

## Column mapping for `whatsapp` sheet

Map column INDICES to field names (zero-indexed):

| Index | Letter | Field |
|-------|--------|-------|
| 0 | A | phone |
| 1 | B | profile_name |
| 2 | C | language |
| 3 | D | current_step |
| 4 | E | service_chosen |
| 5 | F | subservice |
| 6 | G | collected_name |
| 7 | H | collected_website |
| 8 | I | collected_email |
| 9 | J | collected_phone |
| 10 | K | first_message_at |
| 11 | L | last_message_at |
| 12 | M | booking_url_sent |
| 13 | N | booking_confirmed |
| 14 | O | freechat_history |

When accessing in Make IML:
- `{{10.A}}` = phone
- `{{10.B}}` = profile_name
- `{{10.D}}` = current_step (THIS replaces 10.current_step in old broken schema)
- `{{10.L}}` = last_message_at

---

## TASK for Cursor agent

1. **Re-write** W6_blueprint_current.json using EXACT schemas above for:
   - All Sheets searchRows / addRow / updateRow modules
   - All Router filter conditions
2. Replace ALL `{{10.field_name}}` with `{{10.A}}`, `{{10.D}}` etc. (column letters)
3. Replace `{{10.__ROW_NUMBER__}}` correctly when checking session existence
4. Keep HTTP modules and WebhookRespond UNCHANGED (those work)
5. Open new PR with corrected blueprint
6. Verify with `python3 -m json.tool` that JSON is valid
