# CheckPoint — SL CE - Flaca Agent

> Aggiornato: 2026-07-28 | Versione live: **v1.0.0.56** (revert da v1.0.0.59)

---

## ✅ Cosa è stato fatto

### Fase 1 — Menu giornaliero
- Trigger schedulato 09:00 (lun-ven), blocco weekend
- Flow `ScriviMenuSuSharePoint`: scrive piatti su FlacaMenu con Indice progressivo
- Flow `InvioMenuTeams`: invia menu formattato nel gruppo Teams

### Fase 2 — Raccolta ordini
- Topic `RaccoltaOrdini` in Copilot Studio
- Flow `ValidaOrdine`: algoritmo free-text in 3 fasi (estrazione numero, cerca per Indice, contains su titoli)
- Flow `RegistraOrdine`: crea/aggiorna ordine in FlacaOrdini con deduplicazione utente+data
- Check orario (10:00–18:00 test) gestito nel flow PA con DST automatico
- DisplayName reale via O365 Users connector
- Fix FlowActionException: response key `text` allineata al botcomponent CS

### Copilot Studio — Strumenti
- GPT instructions compatte (~550 char)
- `modelDescription` corretta su tutti e 4 gli strumenti (campo AI routing)
- `<description>` nei `botcomponent.xml` (campo UI Copilot Studio)
- Descriptions sui parametri input di ogni action

### Infrastruttura
- SharePoint Lists: `FlacaMenu`, `FlacaOrdini`
- Connection references: SharePoint + O365 Users
- Repository Git aggiornato, CHANGELOG e CheckPoint presenti
- ALM: pack/unpack/import con PAC CLI

---

## ⏳ Cosa manca ancora

### Feature — Ordini multipli stesso utente (progettata, da reimplementare)
- [ ] Tentativo implementazione v1.0.0.57-59 fallito — revert a v1.0.0.56
- [ ] La logica è corretta ma il topic CS ha problemi di gestione del flusso conversazionale (question saltate, variabili vuote, messaggi in ritardo)
- [ ] Da riprogettare con approccio più semplice: valutare se usare un topic separato per "ordina per altri" invece di ramificazioni inline
- [ ] Comportamento atteso:
  - Utente ordina di nuovo → agente chiede "Aggiorna il tuo ordine o ordini per qualcun altro?"
  - Se aggiorna → sovrascrive ordine esistente
  - Se per altri → chiede nome → crea nuovo ordine con quel nome

### Fase 3 — Chiusura ordini
- [ ] Flow `ChiusuraOrdini` schedulato alle 12:00 (aggregazione + notifica Teams)
- [ ] Ripristinare orario da 18:00 → 12:00 dopo fine test

### Fase 4 — Invio ordine WhatsApp
- [ ] Definire connettore WhatsApp (Business API o alternativa)
- [ ] Flow `InvioWhatsApp`: formatta riepilogo, invia, gestisce errore

### Fase 5 — Conferma ristorante
- [ ] Gestione risposta WhatsApp → stato `confermato` + notifica Teams

---

## 🚧 Blocchi / Decisioni aperte

- **Feature ordini multipli**: implementazione fallita (v1.0.0.57-59), revert a v1.0.0.56. Da riprogettare nella prossima sessione con approccio diverso (topic separato vs inline)
- **WhatsApp connector**: da decidere quale servizio usare (Business API, Twilio, altro)
- **Orario chiusura**: attualmente 18:00 per test — da riportare a 12:00 prima del go-live

---

## 📚 Pattern appresi in questa sessione

- **`Contains()` non esiste in CS Power Fx** → usare `sub in str`
- **`alwaysPrompt: true`** obbligatorio su Question nodes quando la variabile potrebbe essere pre-inizializzata
- **Variabili CS init a `Text("")`** bloccano i Question nodes che le usano → non inizializzare le variabili che saranno target di domande
- **Mapping campi botcomponent**: `modelDescription` (AI routing) ≠ `description` (UI) ≠ `<description>` in XML


---

## ✅ Cosa è stato fatto

### Fase 1 — Menu giornaliero
- Trigger schedulato 09:00 (lun-ven), blocco weekend
- Flow `ScriviMenuSuSharePoint`: scrive piatti su FlacaMenu con Indice progressivo
- Flow `InvioMenuTeams`: invia menu formattato nel gruppo Teams

### Fase 2 — Raccolta ordini
- Topic `RaccoltaOrdini` in Copilot Studio
- Flow `ValidaOrdine`: algoritmo free-text in 3 fasi (estrazione numero, cerca per Indice, contains su titoli)
- Flow `RegistraOrdine`: crea/aggiorna ordine in FlacaOrdini con deduplicazione utente+data
- Check orario (10:00–18:00 test) gestito nel flow PA con DST automatico
- DisplayName reale via O365 Users connector
- Fix FlowActionException: response key `text` allineata al botcomponent CS

### Copilot Studio — Strumenti
- GPT instructions compatte (~550 char)
- `modelDescription` corretta su tutti e 4 gli strumenti (campo AI routing)
- `<description>` nei `botcomponent.xml` (campo UI Copilot Studio)
- Descriptions sui parametri input di ogni action

### Infrastruttura
- SharePoint Lists: `FlacaMenu`, `FlacaOrdini`
- Connection references: SharePoint + O365 Users
- Repository Git aggiornato, CHANGELOG e CheckPoint presenti
- ALM: pack/unpack/import con PAC CLI

---

## ⏳ Cosa manca ancora

### Fase 3 — Chiusura ordini
- [ ] Flow `ChiusuraOrdini` schedulato alle 12:00 (aggregazione + notifica Teams)
- [ ] Ripristinare orario da 18:00 → 12:00 dopo fine test

### Fase 4 — Invio ordine WhatsApp
- [ ] Definire connettore WhatsApp (Business API o alternativa)
- [ ] Flow `InvioWhatsApp`: formatta riepilogo, invia, gestisce errore

### Fase 5 — Conferma ristorante
- [ ] Gestione risposta WhatsApp → stato `confermato` + notifica Teams

---

## 🚧 Blocchi / Decisioni aperte

- **WhatsApp connector**: da decidere quale servizio usare (Business API, Twilio, altro)
- **Orario chiusura**: attualmente 18:00 per test — da riportare a 12:00 prima del go-live
- **Test v1.0.0.56 su Teams**: verificare che il nuovo ValidaOrdine gestisca correttamente il testo libero
