# CheckPoint — SL CE - Flaca Agent

> Aggiornato: 2026-10-20 | Versione live: **v1.0.0.73** (ambiente: CopilotStudio_Playground)

---

## ✅ Cosa è stato fatto

### Fase 1 — Menu giornaliero
- Flow `ActionInvioMenuTeams`: legge il menu del giorno da SharePoint (`FlacaMenu`, per categoria: Primi, Secondi, Pizze, Insalate, Burger) e compone il messaggio Markdown
- **Trigger schedulato proprio** (`Recurrence`, blocco weekend incluso): il flow pubblica **direttamente** su Teams (connettore `shared_teams`, `PostMessageToConversation`) — non passa più dall'orchestrazione Copilot Studio
- Verificato funzionante in produzione (10/09/2026)
- Orario in CRM: **10:00** (lun-ven)

### Fase 2 — Raccolta ordini
- Topic `TopicRaccoltaOrdini`: routing esclusivo via istruzioni GPT esplicite (nessuna `triggerQueries`)
- Flow `ActionValidaOrdine`: algoritmo free-text in 3 fasi (estrazione numero, ricerca per Indice, contains su titoli)
- Flow `ActionRegistraOrdine`: crea/aggiorna ordine in `FlacaOrdini` con deduplicazione utente+data
- Gestione "piatto fuori menu": conferma con Adaptive Card Sì/No, poi registrazione con testo libero
- Check orario runtime nel flow PA (con gestione DST)
- DisplayName reale via connettore O365 Users
- Dopo ogni operazione riuscita, invocazione di `ActionPubblicaTabellaOrdini` → pubblicazione tabella ordini aggiornata su Teams

### Fase 2b — Annullamento ordine
- Topic `TopicAnnullaOrdine` + flow `ActionAnnullaOrdine`: verifica orario, cerca ordine utente per DisplayName, elimina se presente
- Esiti gestiti: `annullato` / `non_trovato` / `fuori_orario`
- Pubblicazione tabella ordini aggiornata dopo annullamento riuscito

### Fase 2c — Ordine per conto di terzi (validata 20/10/2026)
- Topic `TopicOrdinaPerAltri`: chiede nome beneficiario e piatto, valida con `ValidaOrdine`, registra con `RegistraOrdine` passando il nome libero come chiave utente (fallback già presente nei flow, nessuna modifica ai flow), pubblica la tabella; gestione fuori menu con conferma Sì/No
- Routing in `gpt.default` ("ORDINE PER CONTO DI ALTRI"); `TopicRaccoltaOrdini` invariato
- Limiti: solo ordini (non annullamento per terzi); se il nome libero non coincide col display name Teams della persona nasce una riga separata

### Fase 3 — Chiusura ordini (validata 20/10/2026)
- Flow `ActionChiusuraOrdini` + `ActionTriggerChiusuraOrdini` (Recurrence 13:00, tutti i giorni) + topic `TopicChiusuraOrdini`
- Finestra ordini **10:00-13:00** allineata in `ActionRegistraOrdine`, `ActionAnnullaOrdine`, messaggio menu e `gpt.default`
- Test in produzione superato, recapito Teams verificato, nessun bug dell'orchestratore riscontrato nei test

### Architettura Copilot Studio
- Recognizer bot-level: `GenerativeAIRecognizer` + `GenerativeActionsEnabled: true` (entrambi attivi contemporaneamente)
- Routing basato su istruzioni esplicite in `gpt.default` verso i topic dedicati (`TopicRaccoltaOrdini`, `TopicAnnullaOrdine`, `TopicResetMenuGiornaliero`)
- **Eccezione documentata**: `ActionInvioMenuTeams` non passa più dall'agente (vedi Fase 1) — per evitare che il messaggio menu (contenente nomi di piatti) venga reinterpretato come input utente e causi un loop, il messaggio è prefissato dal marcatore `🍽️ MENU DEL GIORNO — La Flaca` e `gpt.default` istruisce l'orchestratore a ignorarlo completamente se ricevuto in ingresso
- Topic `TopicInvioMenuTeams` e flow `ActionTriggerInvioMenu` rimossi dal CRM (non più necessari col nuovo pattern)

### Infrastruttura
- SharePoint Lists: `FlacaMenu`, `FlacaOrdini`
- Connection references: SharePoint online, O365 Users, Teams (`shared_teams_1`)
- Repository Git aggiornato e sincronizzato con lo stato reale del CRM
- ALM: pack/unpack/import/export con PAC CLI (ambiente CopilotStudio_Playground)

---

## ⏳ Cosa manca ancora

### Fase 1b — Richiesta menu su richiesta utente (TODO)
- [ ] Quando l'utente chiede il menu, GPT deve triggerare il flusso di invio menu su Teams. Analisi fatta (20/10/2026): `ActionInvioMenuTeams` ha solo trigger `Recurrence` (non invocabile da topic, un flow ha un solo trigger) e termina nel weekend; `TopicInvioMenuTeams` non esiste più; in `gpt.default` c'è solo lo starter `INVIA_MENU_TEAMS` senza regola di routing
- [ ] **Opzione A (consigliata)**: nuovo flow `ActionInvioMenuTeamsManuale` (copia con trigger agente, senza guardia weekend) + nuovo `TopicMenuGiornaliero` (conferma "✅ Menu inviato nel gruppo") + regola di routing in `gpt.default`. Flow schedulato invariato; svantaggio: logica menu duplicata
- [ ] Opzione B: flow unico col trigger agente + scheduler sottile che lo chiama come child flow (nessuna duplicazione, ma compatibilità trigger agente/child flow da verificare e rischio sul menu delle 10:00)
- [ ] Decisioni da prendere: A o B; richiesta manuale valida anche nel weekend; richiesta manuale accettata anche fuori dalla finestra 10:00-13:00 o rifiutata come "ordini chiusi"

### Fase 2b — Annullamento per conto di terzi
- [ ] Valutare `TopicAnnullaPerAltri` (stesso approccio: nome libero come chiave utente)

### Fase 4 — Invio ordine WhatsApp
- [ ] Definire connettore WhatsApp (Business API o alternativa)
- [ ] Flow `InvioWhatsApp`: formatta riepilogo, invia, gestisce errore

### Fase 5 — Conferma ristorante
- [ ] Gestione risposta WhatsApp → stato `confermato` + notifica Teams

### Altro
- [ ] Dopo ogni import verificare che i flow con Recurrence (`ActionInvioMenuTeams`, `ActionTriggerChiusuraOrdini`) siano ON

---

## 🚧 Blocchi / Decisioni aperte

- **WhatsApp connector**: da decidere quale servizio usare (Business API, Twilio, altro)
- **Richiesta menu da GPT**: scegliere Opzione A o B (vedi Fase 1b)
- **Trigger chiusura nel weekend**: gira tutti i giorni (nessun blocco weekend, a differenza del menu); lasciato così per ora

---

## 📚 Pattern appresi

- **`Contains()` non esiste in CS Power Fx** → usare `sub in str`
- **`alwaysPrompt: true`** obbligatorio su Question nodes quando la variabile potrebbe essere pre-inizializzata
- **Variabili CS init a `Text("")`** bloccano i Question nodes che le usano → non inizializzare le variabili target di domande
- **Mapping campi botcomponent**: `modelDescription` (AI routing) ≠ `description` (UI) ≠ `<description>` in XML
- **`ExecuteCopilot` da Power Automate ≠ canale Teams**: una conversazione avviata da flow esterno tramite il connettore Copilot Studio è isolata/effimera e **non recapita automaticamente** i messaggi del topic nel canale Teams reale. Per flow schedulati che devono pubblicare su un canale Teams fisso, il pattern corretto è il post diretto (`shared_teams`), non l'invocazione dell'agente
- **Rischio loop con post diretto su Teams**: qualunque messaggio postato da un flow con `shared_teams` viene ri-letto dal bot come nuovo input. Se il contenuto assomiglia a un ordine (contiene nomi di piatti, come il menu), va protetto con un marcatore riconoscibile e una guardia esplicita nelle istruzioni GPT che lo ignori
- **Recurrence + import**: i flow con trigger `Recurrence` vengono talvolta importati disattivati — verificare sempre lo stato ON dopo l'import
