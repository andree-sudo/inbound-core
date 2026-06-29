# Inbound Core — n8n Workflow

🇬🇧 [English](README.md) · 🇺🇦 [Українська](README.uk.md)

---

### Prehľad

**Inbound Core** je automatizačný workflow pre n8n, ktorý spracúva prichádzajúce e-maily, hodnotí ich obchodnú relevanciu pomocou AI modelu (Google Gemini) a — v prípade relevantných správ — automaticky vytvára dokument s obchodnou ponukou, ukladá kontaktné a cenové údaje do Google Sheets a zaznamenáva výsledok.

---

### Ako to funguje

```
Gmail Trigger
     │
     ▼
Google Gemini (AI Intake Assistant)
     │
     ▼
Code in JavaScript1 (spracovanie odpovede AI)
     │
     ▼
If — Je relevantný?
 ├── NIE → Odoslanie upozornenia adminovi
 └── ÁNO → Vytvorenie Google Docs (Ponuka)
               │
               ▼
           Aktualizácia dokumentu (obsah ponuky)
               │
               ▼
           Code in JavaScript (extrakcia doc_id)
               │
               ▼
           Pridanie / aktualizácia riadku v Google Sheets (log)
               │
               ▼
           Aktualizácia dokumentu (finálna suma)
```

---

### Popis uzlov

| Uzol | Typ | Účel |
|------|-----|------|
| **Gmail Trigger** | Trigger | Sleduje doručenú poštu každú minútu |
| **Message a model** | Google Gemini | Analyzuje e-mail: hodnotí relevanciu, extrahuje kontaktné údaje, určuje kategóriu služby, načítava ceny |
| **Code in JavaScript1** | Kód | Spracúva JSON odpoveď AI; označuje nerelevantné e-maily |
| **If** | Podmienka | Rozdeľuje tok: relevantný vs. nerelevantný |
| **Send a message1** | Gmail | Odosiela upozornenie adminovi, ak e-mail nie je možné spracovať |
| **Create a document** | Google Docs | Vytvorí nový dokument s názvom `Proposal — {Meno klienta}` |
| **Update a document** | Google Docs | Vloží obsah ponuky (info o klientovi, služba, základná cena) |
| **Code in JavaScript** | Kód | Extrahuje ID nového dokumentu pre ďalšie úpravy |
| **Append or update row in sheet** | Google Sheets | Zaznamená vypočítaný výsledok a detaily spracovania (List 3) |
| **Update a document1** | Google Docs | Doplní finálnu sumu a dátum výpočtu do ponuky |
| **Google_Sheets_Add_Contact** | AI nástroj | Uloží kontakt do Listu 1 |
| **Google_Sheets_Add_Calculation_Input** | AI nástroj | Uloží vstupné dáta výpočtu do Listu 2 |
| **Google_Sheets_Get_Pricing** | AI nástroj | Načíta cenové dáta pre zistenú kategóriu služby |

---

### Podporované kategórie služieb

| Služba | category_id |
|--------|-------------|
| Cloud Migration | `srv_cloud_mig` |
| Data Pipeline Setup | `srv_data_eng` |
| AI Architecture Audit | `srv_ai_audit` |
| Custom Backend Development | `srv_custom_dev` |
| Legacy System Support | `srv_legacy_supp` |

---

### Pravidlá relevancie e-mailov

**Relevantný** — odosielateľ:
- Žiada o cenovú ponuku alebo konzultáciu
- Chce si objednať službu
- Diskutuje o novom projekte
- Prejavuje záujem o niektorú z podporovaných služieb

**Nerelevantný** — e-mail je:
- Spam alebo marketing
- Žiadosť o prácu alebo životopis
- Osobná správa
- Automatické systémové oznámenie

---

### Požiadavky a konfigurácia

Pred aktiváciou workflowu nahraďte všetky zástupné hodnoty v JSON súbore:

| Zástupná hodnota | Čo doplniť |
|------------------|------------|
| `YOUR_CREDENTIAL_ID` | ID prihlasovacích údajov v n8n pre každú službu |
| `YOUR_SPREADSHEET_ID` | ID tabuľky Google Sheets |
| `YOUR_SPREADSHEET_URL` | Plná URL tabuľky |
| `YOUR_FOLDER_ID` | ID priečinka Google Drive pre ponuky |
| `YOUR_EMAIL` | E-mailová adresa administrátora pre upozornenia |
| `YOUR_WEBHOOK_ID` | n8n webhook ID (ak sa používa) |
| `YOUR_INSTANCE_ID` | ID vašej n8n inštancie |

**Požadované prihlasovanie v n8n:**
- Google Gemini (PaLM) API
- Google Sheets OAuth2
- Google Docs OAuth2
- Gmail OAuth2

**Štruktúra Google Sheets:**
- **List 1 (`Аркуш1`)** — Kontakty (`user_id`, `full_name`, `email`, `phone_number`, `status`)
- **List 2** — Vstupné dáta výpočtu (`category_id`, `input_value_1`, `input_value_2`, `raw_email_body`)
- **List 3 (`Аркуш3`)** — Log spracovania (`calculated_result`, `created_at`, `processing_log`)

---

### Poznámky

- Workflow je predvolene nastavený ako **neaktívny** — aktivujte ho manuálne po konfigurácii.
- Gmail Trigger sa spúšťa **každú minútu**; frekvenciu je možné upraviť podľa potreby.
- Ak odpoveď AI modelu nie je možné spracovať (neplatný JSON), flow zlyhá bez upozornenia — zvážte pridanie uzla na spracovanie chýb.
- Telefónne čísla sa ukladajú **bez** predpony `+`.
- Ak sa v e-maile nenájde meno, kontakt sa uloží ako `Unknown`.
