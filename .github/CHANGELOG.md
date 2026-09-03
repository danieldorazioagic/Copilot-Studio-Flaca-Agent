# CHANGELOG — SL CE - Flaca Agent

## [1.0.0.60] - 2026-09-03
### Modifiche
- **Fix critico**: risolto mismatch case-sensitive delle variabili di output nei topic, introdotto dal re-salvataggio via UI Copilot Studio (necessario per riparare il riferimento CloudFlow di `TopicAnnullaOrdine`)
  - `TopicRaccoltaOrdini`: l'azione `ValidaOrdine` scriveva in `Topic.nomePiatto`/`Topic.numeroPiatto` mentre il resto del topic leggeva `Topic.NomePiatto`/`Topic.NumeroPiatto` → **bloccava tutta la raccolta ordini** (ogni piatto risultava "non trovato")
  - `TopicAnnullaOrdine`: l'azione `AnnullaOrdine` scriveva in `Topic.esito` mentre il `ConditionGroup` leggeva `Topic.Esito` → nessun messaggio di esito mostrato
  - Allineati tutti i riferimenti al case effettivo del binding
- Sincronizzata la cartella `SLCEFlacaAgent_unpacked_crm` con lo stato reale del CRM dopo il fix manuale in Copilot Studio Maker Portal (i flow sono stati rinominati con prefisso `Action` durante l'operazione)

## [1.0.0.59] - 2026-09-03
### Modifiche
- **Fix**: aggiunte le action botcomponent mancanti `ActionAnnullaOrdine` e `ActionPubblicaTabellaOrdini` (disattivate, stesso pattern delle altre 4 action), necessarie perché Copilot Studio risolvesse correttamente i riferimenti CloudFlow nei topic (errore `InvalidReferenceError` / `CloudFlow ... not found` su `TopicAnnullaOrdine`)

## [1.0.0.58] - 2026-09-03
### Modifiche
- Nuovo flow `AnnullaOrdine`: verifica orario, cerca ordine utente per DisplayName in FlacaOrdini, elimina la riga se presente. Esiti: `annullato` / `non_trovato` / `fuori_orario`
- Nuovo flow `PubblicaTabellaOrdini`: legge tutti gli ordini del giorno da FlacaOrdini e pubblica su Teams una tabella testuale Utente → Piatto (o messaggio "nessun ordine" se vuota)
- Nuovo topic `TopicAnnullaOrdine`: unico punto di ingresso per l'annullamento ordine, invoca il flow `AnnullaOrdine` e, in caso di successo, invoca `PubblicaTabellaOrdini`
- Aggiornato `TopicRaccoltaOrdini`: dopo esito `nuovo`/`aggiornato`, invoca `PubblicaTabellaOrdini` per pubblicare il riepilogo aggiornato su Teams
- Aggiornate GPT instructions: regola di routing per l'intento di annullamento ordine verso `TopicAnnullaOrdine`
- Registrati in `Solution.xml` i 2 nuovi flow come RootComponent (type 29)

## [1.0.0.57] - 2026-09-03
### Modifiche
- Rename friendly name solution: "SL CE - Flaca Agent" → "SLCEFlacaAgent" (allineato allo Unique Name)
- Naming uniformato su tutti i componenti bot: convenzione `SL CE - Flaca Agent | <Tipo><Nome>` (es. `TopicRaccoltaOrdini`, `ActionValidaOrdine`)
- Valorizzate tutte le description mancanti (es. `TopicRaccoltaOrdini`)
- **Fix architetturale**: disattivate (statecode/statuscode) le 4 Action standalone (`ValidaOrdine`, `RegistraOrdine`, `ScriviMenuSuSharePoint`, `CancellaMenuOggi`) esposte come tool diretti all'orchestratore AI — causa nota di comportamento incoerente su Teams (bypass dei topic)
- Aggiunto nuovo topic `TopicResetMenuGiornaliero`: wrapper con conferma di sicurezza per il flow `CancellaMenuOggi`, unico punto di ingresso per il reset admin (sostituisce l'esposizione diretta del tool)
- Aggiornate GPT instructions: regola architetturale "solo topic" (mai invocare flow/tool direttamente), riferimenti aggiornati al nuovo naming dei topic

## [1.0.0.56] - 2026-07-28
### Modifiche
- Fix `modelDescription` ValidaOrdine: era erroneamente impostata con la descrizione di RegistraOrdine
- Appreso e documentato pattern obbligatorio struttura file `data` action CS: `modelDescription` (AI routing) vs `description` (UI estesa)

## [1.0.0.55] - 2026-07-28
### Modifiche
- Aggiunta `<description>` in tutti e 4 i `botcomponent.xml` delle action (ValidaOrdine, RegistraOrdine, CancellaMenuOggi, ScriviMenuSuSharePoint) — campo UI "Descrizione" in Copilot Studio
- Acquisito da UI CS il campo `modelDescription` come campo corretto per routing AI orchestratore

## [1.0.0.54] - 2026-07-28
### Modifiche
- Riscrittura completa flow `ValidaOrdine`: nuovo algoritmo free-text in 3 fasi (estrazione numero da parole, cerca per Indice, contains su tutti i piatti)
- Gestisce input come "vorrei un 32" o "oggi voglio Fiori e alici"
- Pulizia GPT instructions: rimossa sezione ESTRAZIONE INPUT ORDINE (inutile con fix al flow)
- Corretta `description` parametro `piatto` di ValidaOrdine (ora accurata)

## [1.0.0.53] - 2026-07-28
### Modifiche
- Tentativo fix estrazione input via GPT instructions + action description (approccio superato da v1.0.0.54)

## [1.0.0.52] - 2026-07-28
### Modifiche
- GPT instructions: ridotte da ~3.500 a ~550 caratteri, rimosso dettaglio operativo ridondante
- Aggiunta `description` a tutte e 4 le action (ValidaOrdine, RegistraOrdine, CancellaMenuOggi, ScriviMenuSuSharePoint)
- Aggiunta `description` a tutti i parametri input delle action per supporto orchestratore AI

## [1.0.0.51] - 2026-07-28
### Modifiche
- Fix FlowActionException: rinominata chiave response `esito` → `text` nel nodo `Respond_to_the_agent` del flow `RegistraOrdine` per allineamento con botcomponent outputType CS

## [1.0.0.50] - 2026-07-07
### Modifiche
- Fix OData filter: apostrofi nel DisplayName escapati con `replace()` in `RegistraOrdine`

## [1.0.0.49] - 2026-07-07
### Modifiche
- Check orario delegato interamente al flow PA (`W. Europe Standard Time`, DST automatico)
- Binding esito ripristinato nel topic `RaccoltaOrdini`

## [1.0.0.48] - 2026-07-07
### Modifiche
- Orario chiusura ordini spostato a 18:00 per fase di test (da 12:00)

## [1.0.0.46] - 2026-07-07
### Modifiche
- Fix GPT Instructions: contesto gruppo Teams multi-utente esplicito
- Fix topic: trigger queries aggiornate, uso `System.User.PrincipalName`

## [1.0.0.44] - 2026-07-06
### Modifiche
- Aggiunto check orario nel flow `RegistraOrdine`
- Integrazione O365 Users connector per recupero DisplayName reale

## [1.0.0.42] - 2026-07-05
### Modifiche
- Fix piatto: letto da FlacaMenu via Indice (non da CS AI)

## [1.0.0.39] - 2026-07-04
### Modifiche
- Fix Indice in `ScriviMenuSuSharePoint`: cast Float→Int con `int()` 

## [1.0.0.36] - 2026-07-03
### Modifiche
- Blocco sabato/domenica in flow `Trigger` e `InvioMenuTeams`
