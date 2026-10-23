# CheckPoint — SL CE - Flaca Agent

> Aggiornato: 2026-10-23 | Versione live: **v1.0.0.81** (ambiente: CopilotStudio_Playground)

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

### Fase 2c — Ordine e annullamento per conto di terzi (v1.0.0.74, in uso)
- `TopicOrdinaPerAltri` **rimosso**: il soggetto è riconosciuto da un prompt AI Builder (`PromptRiconosciSoggetto`, input `UserRequest`, output `ME` oppure il nome) invocato dentro `TopicRaccoltaOrdini` e `TopicAnnullaOrdine`
- Il nome ottenuto valorizza `Topic.UtenteUPN`; i flow `RegistraOrdine` e `AnnullaOrdine` risolvono il nome via O365 con fallback sul testo libero (nessuna modifica ai flow)
- Limiti: se il nome libero non coincide col display name Teams nasce una riga separata e l'annullamento non la trova

### Fase 2d — Riepilogo ordini su richiesta (v1.0.0.75-77)
- Flow `ActionRiepilogoOrdini` + `TopicRiepilogaOrdini` (creato da UI per generare l'associazione topic↔flow): quantità per piatto e totale, sola consultazione, nessun vincolo orario

### Fase 1b — Menu su richiesta (v1.0.0.78)
- Flow `ActionMostraMenu` (trigger agente, senza blocco weekend) + `TopicMostraMenu`: mostra il menu di oggi con la stessa numerazione del menu delle 10:00; messaggio ⚠️ se non disponibile
- Regola MOSTRA MENU in `gpt.default`; se il messaggio contiene un piatto resta `TopicRaccoltaOrdini`
- Sostituisce le Opzioni A/B analizzate in precedenza (flow dedicato, senza duplicare la logica nel flow schedulato)

### Fase 1c — Come funziona (v1.0.0.77)
- `TopicComeFunziona` su richiesta ("come funziona", "aiuto") e messaggio automatico `ℹ️ COME FUNZIONA — La Flaca` dopo il menu (azione `Invia_Istruzioni_Su_Teams` in `ActionInvioMenuTeams`), ignorato dal bot tramite guardia in `gpt.default`
- Riscritte le descrizioni di tutti i topic e flow; `CancelAllDialogs` in coda ai topic

### Fix e messaggi (v1.0.0.79-81)
- Menu (`ActionInvioMenuTeams`, `ActionMostraMenu`): numero in grassetto (`**8.** Titolo`) e a capo doppio dopo ogni piatto. Le liste Markdown non partenti da 1 da mobile venivano mostrate una lettera per riga; un solo a capo univa i piatti (verificato da desktop e telefono)
- `topic.Escalate`: messaggio personalizzato con `System.User.DisplayName` e apertura di `TopicComeFunziona`. Attenzione: `Fallback` e `EndofConversation` richiamano `Escalate`, quindi lo stesso messaggio compare anche dopo input non compresi

### Fase 3 — Chiusura ordini (validata 20/10/2026)
- Flow `ActionChiusuraOrdini` + `ActionTriggerChiusuraOrdini` (Recurrence 13:00, tutti i giorni) + topic `TopicChiusuraOrdini`
- Finestra ordini **10:00-13:00** allineata in `ActionRegistraOrdine`, `ActionAnnullaOrdine`, messaggio menu e `gpt.default`
- Test in produzione superato, recapito Teams verificato, nessun bug dell'orchestratore riscontrato nei test

### Architettura Copilot Studio
- Recognizer bot-level: `GenerativeAIRecognizer` + `GenerativeActionsEnabled: true` (entrambi attivi contemporaneamente)
- Routing basato su istruzioni esplicite in `gpt.default` verso i topic dedicati (`TopicRaccoltaOrdini`, `TopicAnnullaOrdine`, `TopicRiepilogaOrdini`, `TopicMostraMenu`, `TopicComeFunziona`, `TopicResetMenuGiornaliero`)
- **Eccezione documentata**: `ActionInvioMenuTeams` non passa più dall'agente (vedi Fase 1) — per evitare che il messaggio menu (contenente nomi di piatti) venga reinterpretato come input utente e causi un loop, il messaggio è prefissato dal marcatore `🍽️ MENU DEL GIORNO — La Flaca` e `gpt.default` istruisce l'orchestratore a ignorarlo completamente se ricevuto in ingresso
- Topic `TopicInvioMenuTeams` e flow `ActionTriggerInvioMenu` rimossi dal CRM (non più necessari col nuovo pattern)

### Infrastruttura
- SharePoint Lists: `FlacaMenu`, `FlacaOrdini`
- Connection references: SharePoint online, O365 Users, Teams (`shared_teams_1`)
- Repository Git aggiornato e sincronizzato con lo stato reale del CRM
- ALM: pack/unpack/import/export con PAC CLI (ambiente CopilotStudio_Playground)

---

## ⏳ Cosa manca ancora

### Da testare in Teams (v1.0.0.81)
- [x] `TopicMostraMenu`, `TopicComeFunziona`, riepilogo, `Escalate` confermati dall'utente il 23/10
- [ ] Numero 9 mancante nel menu (visto in uno screenshot): controllare `FlacaMenu`
- [ ] Prompt soggetto: ordine/annullamento per sé e per un collega taggato (non testato a runtime in modo sistematico)

### Pulizia
- [ ] Componente `action.SLCE-FlacaAgentRiepilogoOrdini` inutilizzato ancora nel CRM e nel repo (richiede bump + import)

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
- **Trigger chiusura nel weekend**: il flow `ActionTriggerChiusuraOrdini` salta il weekend (lun-ven alle 13:00)
- **Limite flow**: lettura massima di 200 ordini

---

## 📚 Pattern appresi

- **`Contains()` non esiste in CS Power Fx** → usare `sub in str`
- **`alwaysPrompt: true`** obbligatorio su Question nodes quando la variabile potrebbe essere pre-inizializzata
- **Variabili CS init a `Text("")`** bloccano i Question nodes che le usano → non inizializzare le variabili target di domande
- **Mapping campi botcomponent**: `modelDescription` (AI routing) ≠ `description` (UI) ≠ `<description>` in XML
- **`ExecuteCopilot` da Power Automate ≠ canale Teams**: una conversazione avviata da flow esterno tramite il connettore Copilot Studio è isolata/effimera e **non recapita automaticamente** i messaggi del topic nel canale Teams reale. Per flow schedulati che devono pubblicare su un canale Teams fisso, il pattern corretto è il post diretto (`shared_teams`), non l'invocazione dell'agente
- **Rischio loop con post diretto su Teams**: qualunque messaggio postato da un flow con `shared_teams` viene ri-letto dal bot come nuovo input. Se il contenuto assomiglia a un ordine (contiene nomi di piatti, come il menu), va protetto con un marcatore riconoscibile e una guardia esplicita nelle istruzioni GPT che lo ignori
- **Topic che invoca un flow**: serve la riga in `Assets/botcomponent_workflowset.xml` (senza, `CloudFlow NotFound`); un nuovo flow richiede JSON, `.json.data.xml` e `RootComponent type="29"` in `Solution.xml`
- **Profilo pac**: verificare con `pac auth list` prima di ogni import (può passare a un altro ambiente)
- **Recurrence + import**: i flow con trigger `Recurrence` vengono talvolta importati disattivati — verificare sempre lo stato ON dopo l'import
