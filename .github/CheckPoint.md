# CheckPoint — SL CE - Flaca Agent

> Aggiornato: 2026-07-28 | Versione live: **v1.0.0.51**

---

## ✅ Cosa è stato fatto

### Fase 1 — Menu giornaliero
- Trigger schedulato 09:00 (lun-ven), blocco weekend
- Flow `ScriviMenuSuSharePoint`: scrive piatti su FlacaMenu con Indice progressivo
- Flow `InvioMenuTeams`: invia menu formattato nel gruppo Teams

### Fase 2 — Raccolta ordini
- Topic `RaccoltaOrdini` in Copilot Studio
- Flow `ValidaOrdine`: cerca piatto per Indice o Nome su FlacaMenu
- Flow `RegistraOrdine`: crea/aggiorna ordine in FlacaOrdini con deduplicazione utente+data
- Check orario (10:00–18:00 test / 12:00 prod) gestito nel flow PA con DST automatico
- DisplayName reale via O365 Users connector
- Fix OData filter apostrofi nel DisplayName
- Fix binding output `RegistraOrdine`: response key `text` allineata al botcomponent CS ✅ (v1.0.0.51)

### Infrastruttura
- SharePoint Lists: `FlacaMenu`, `FlacaOrdini`
- Connection references: SharePoint + O365 Users
- Repository Git configurato e aggiornato
- ALM: pack/unpack/import con PAC CLI

---

## ⏳ Cosa manca ancora

### 🔴 Priorità alta — Test v1.0.0.51
- [ ] Testare su Teams: scrivi nome piatto → `✅ Ordine registrato con successo!`
- [ ] Testare aggiornamento ordine → `✅ Ordine aggiornato con successo!`
- [ ] Testare fuori orario → `⚠️ Gli ordini di oggi sono già chiusi.`
- [ ] Verificare DisplayName corretto in FlacaOrdini

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
