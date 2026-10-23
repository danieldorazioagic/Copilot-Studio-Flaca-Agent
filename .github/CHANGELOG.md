# CHANGELOG — SL CE - Flaca Agent

## [1.0.0.78] - 2026-10-23
### Modifiche
- Nuovo flow `ActionMostraMenu`: legge il menu di oggi da FlacaMenu per categoria (stessa numerazione del menu delle 10:00) e restituisce `esito` (ok / non_disponibile) e `menu`
- Nuovo `TopicMostraMenu`: mostra il menu su richiesta, con messaggio ⚠️ se non disponibile; `CancelAllDialogs` finale
- `gpt.default`: regola MOSTRA MENU (se il messaggio contiene un piatto da ordinare resta RaccoltaOrdini)
- `botcomponent_workflowset.xml`: associazione topic↔flow per MostraMenu

## [1.0.0.77] - 2026-10-23
### Modifiche
- Riscritte le descrizioni di topic e flow (11 flow, trigger inclusi); aggiunto `modelDescription` a RaccoltaOrdini e AnnullaOrdine
- Nuovo `TopicComeFunziona` (spiegazione del funzionamento dell'agente) con `CancelAllDialogs` finale
- `ActionInvioMenuTeams`: nuova azione `Invia_Istruzioni_Su_Teams` dopo il menu delle 10:00
- `gpt.default`: regola COME FUNZIONA, marcatore "ℹ️ COME FUNZIONA" ignorato, riferimento corretto a `TopicRiepilogaOrdini`
- `TopicRiepilogaOrdini`: `EndConversation` sostituito da `CancelAllDialogs`

## [1.0.0.76] - 2026-10-23
### Modifiche
- Fix `TopicRiepilogoOrdini`: aggiunto il componente `action.SLCE-FlacaAgentRiepilogoOrdini` che registra il flow `ActionRiepilogoOrdini` sull'agente (errore `InvalidReferenceError` / CloudFlow NotFound)

## [1.0.0.75] - 2026-10-23
### Modifiche
- Nuovo flow `ActionRiepilogoOrdini`: aggrega gli ordini di oggi per piatto (quantità per piatto + totale), sola consultazione, senza cambio di stato né vincolo orario
- Nuovo `TopicRiepilogoOrdini` (routing in `gpt.default`, regola "RIEPILOGO ORDINI"; esclusa la chiusura ordini)
- Aggiunto nodo "Termina tutti gli argomenti" (`CancelAllDialogs`) a `TopicRiepilogoOrdini` e `TopicChiusuraOrdini`
- Sync repo da CRM: prompt `PromptRiconosciSogetto` riscritto (input `UserRequest`) e nodi prompt allineati nei topic `RaccoltaOrdini`/`AnnullaOrdine`

## [1.0.0.74] - 2026-10-23
### Modifiche
- Nuovo prompt AI Builder `PromptRiconosciSogetto` (gpt-4.1-mini): restituisce `ME` oppure il nome della persona taggata
- `TopicRaccoltaOrdini` e `TopicAnnullaOrdine`: nodo prompt che, per ordini/annullamenti per conto di altri, valorizza `Topic.UtenteUPN` con il nome del soggetto
- Rimosso `TopicOrdinaPerAltri` dal repo (da eliminare manualmente nel CRM) e relativa regola da `gpt.default`; ordini/annullamenti per altri instradati ai topic standard
- Sync repo da CRM (aggiornamenti minori a `ActionRegistraOrdine`, `ActionAnnullaOrdine`)

## [1.0.0.73] - 2026-10-20
### Modifiche
- Importata sul Playground (`SLCEFlacaAgent_1_0_0_73.zip`) la sync CRM del 2026-10-20 (di seguito)

## [1.0.0.72] - 2026-10-20 (sync CRM successivo, stessa versione)
### Modifiche
- Sync repo da modifiche manuali in CRM (nessun bump versione, solo allineamento file `SLCEFlacaAgent_unpacked_crm`):
  - `ActionInvioMenuTeams`: orario invio menu da 14:30 a 10:00
  - `ActionRegistraOrdine`: orario di chiusura ordini da 18:00 a 14:00 (finestra ordini 10:00-14:00)
  - `Solution.xml` / `Customizations.xml`: aggiornamento metadati di versione piattaforma
- **Allineamento chiusura ordini alle 13:00 (modifica locale, da importare)**:
  - `ActionRegistraOrdine` e `ActionAnnullaOrdine`: finestra ordini 10:00-13:00 (Annulla era 18:00, Registra 14:00)
  - `ActionTriggerChiusuraOrdini`: già alle 13:00 (invariato)
  - Messaggio menu (`ActionInvioMenuTeams`), `gpt.default` e descrizione action `RegistraOrdine`: testi aggiornati da 12:00 a 13:00
  - Chiusura ordini validata in test
- **Nuovo topic `TopicOrdinaPerAltri`** (modifica locale, da importare): ordine a nome di un'altra persona
  - Chiede nome beneficiario e piatto, valida con `ValidaOrdine`, registra con `RegistraOrdine` passando il nome libero come chiave utente (fallback già presente nel flow, nessuna modifica ai flow), pubblica la tabella con `PubblicaTabellaOrdini`; gestione fuori menu con conferma Si/No
  - `gpt.default`: nuova regola di routing "ORDINE PER CONTO DI ALTRI" verso il nuovo topic; `TopicRaccoltaOrdini` invariato

## [1.0.0.72] - 2026-09-10
### Modifiche
- **Fix critico**: risolto bug per cui il menu giornaliero non veniva mai pubblicato su Teams al trigger schedulato
  - Causa: il trigger `ActionTriggerInvioMenu` invocava l'agente via `ExecuteCopilot` (connettore Microsoft Copilot Studio) da Power Automate, aprendo una conversazione effimera **non legata al canale Teams reale**; il topic rispondeva correttamente ma l'output tornava solo al flow, senza mai raggiungere il gruppo Teams. Il comando manuale funzionava perché digitato dentro una conversazione Teams nativa
  - **Soluzione adottata**: `ActionInvioMenuTeams` torna a pubblicare direttamente su Teams (connettore `shared_teams`, `PostMessageToConversation`), con trigger `Recurrence` proprio (10:00, lun-ven, blocco weekend) invece di passare dall'orchestrazione Copilot Studio
  - Per evitare il rischio di loop noto (bot che rielabora come input utente un messaggio postato da un flow, causa reintrodotta di questo cambio in v1.0.0.67/69), il messaggio menu è ora prefissato dal marcatore `🍽️ MENU DEL GIORNO — La Flaca`; aggiunta guardia esplicita in `gpt.default` che istruisce l'orchestratore a ignorare completamente qualsiasi messaggio in ingresso che inizi con questo marcatore
  - `ActionTriggerInvioMenu` e `TopicInvioMenuTeams` non sono più invocati nel nuovo pattern (lasciati inerti nel repository, gestione a cura dell'utente)
- Nota operativa: verificare che il flow `ActionInvioMenuTeams` sia attivo (ON) in Power Automate dopo l'import, poiché i flow con trigger Recurrence talvolta vengono importati disattivati

## [1.0.0.71] - 2026-09-08 (sync CRM successivo, stessa versione)
### Modifiche
- **Sync repo da modifiche manuali in CRM (nessun bump versione, solo allineamento file)**:
  - `gpt.default`: aggiunti 2 `conversationStarters` — "Richiedi Menu Giornaliero" (→ keyword `INVIA_MENU_TEAMS`) e "Report Ordini" (→ keyword `CHIUDI_ORDINI_GIORNALIERI`)
  - `TopicAnnullaOrdine`: aggiunto nodo `EndConversation` esplicito a fine topic (dopo il `ConditionGroup` di gestione esiti), a rinforzo della prevenzione di doppie risposte/loop
  - `ActionTriggerInvioMenu`: orario Recurrence modificato da `10:00` a `17:15` (verosimilmente per test)
- **Nota aperta**: `ActionTriggerInvioMenu` risulta non avviarsi al momento — da investigare in una sessione successiva (possibili cause: flow disattivato dopo l'ultima modifica, blocco weekend, o orario di test non ancora verificato)

## [1.0.0.71] - 2026-09-08
### Modifiche
- **Fix bug critico**: `TopicInvioMenuTeams` falliva con errore `InvalidReferenceError` / `CloudFlow ... not found` sul flow `ActionInvioMenuTeams`
  - Causa 1 (risolta in questa versione): il file metadata `.json.data.xml` del flow riportava ancora `ModernFlowType=0` (tipo "schedulato"), residuo dalla precedente configurazione a trigger Recurrence. Dopo la conversione del flow in Skill (trigger Request/Skills) in v1.0.0.69, il metadata non era stato aggiornato, impedendo a Copilot Studio di risolvere il flow come CloudFlow invocabile
  - Causa 2 (persistente, risolta manualmente dall'utente in CRM): mancava la voce di associazione topic→flow in `Assets/botcomponent_workflowset.xml` (`ddo_SLCEFlacaAgent.topic.InvioMenuTeams` → `d7372fbf-...`). Questo file viene generato/aggiornato da Copilot Studio solo quando l'azione "Chiama un flusso" viene selezionata/risalvata dall'interfaccia Maker; modificando il flowId direttamente nello YAML del topic (senza passare dal picker UI) questa associazione non viene creata, causando il fallimento della validazione anche con `ModernFlowType` corretto
  - **Nota per il futuro**: quando si crea/modifica un topic che invoca un flow tramite `InvokeFlowAction` editando direttamente lo YAML (senza passare dal Maker), verificare sempre che in `Assets/botcomponent_workflowset.xml` sia presente la voce `<botcomponent_workflow botcomponentid.schemaname="..." workflowid.workflowid="...">` corrispondente; se manca, va aggiunta manualmente oppure il flow va riselezionato una volta dall'interfaccia Maker prima dell'export/import successivo
  - Da tenere a mente anche: ogni volta che un flow viene convertito da Recurrence a Skill (o viceversa), aggiornare anche `ModernFlowType` nel relativo `.json.data.xml`

## [1.0.0.70] - 2026-09-08
### Modifiche
- **Fix sintassi SendActivity nei topic**: corretto il binding delle variabili nei messaggi pubblicati dai topic di sistema. Il formato `activity: =Topic.Variabile` (espressione Power Fx) non valorizza correttamente il messaggio; il formato corretto per interpolare una variabile in un testo è `activity: "{Topic.Variabile}"` (già utilizzato correttamente in `TopicRaccoltaOrdini` a seguito di una modifica manuale in CRM)
  - `TopicAnnullaOrdine`: corretto `msgTabellaAnnulla`
  - `TopicChiusuraOrdini`: corretti `msgChiusura` e `msgRiepilogo`
  - `TopicInvioMenuTeams`: corretto `msgMenu`

## [1.0.0.69] - 2026-09-08
### Modifiche
- **Estensione fix loop Teams a tutti i flussi**: generalizzato il pattern "l'agente scrive, non il flow" a tutti i flussi che pubblicavano su Teams via connettore diretto (`shared_teams`)
  - `ActionPubblicaTabellaOrdini`: corretta la formattazione Markdown (righe separate da `\n\n` invece di `\n` singolo, che veniva collassato da Teams in un'unica riga)
  - `ActionChiusuraOrdini`: rimossa la connessione Teams e i 2 post diretti (notifica chiusura + riepilogo); il flow ora restituisce `messaggioChiusura` e `riepilogo` come testo; il topic `ChiusuraOrdini` pubblica entrambi con 2x `SendActivity`
  - `ActionInvioMenuTeams`: **ristrutturato da flow standalone a Skill invocata dal bot** — prima aveva un proprio trigger Recurrence (10:00) e postava direttamente su Teams, completamente disconnesso dall'orchestrazione Copilot Studio; ora il trigger Recurrence/weekend-check è stato spostato nel nuovo flow `ActionTriggerInvioMenu`, che invoca `ExecuteCopilot` con keyword `INVIA_MENU_TEAMS`; il nuovo topic `TopicInvioMenuTeams` (routing deterministico via `triggerQueries`) invoca il flow come Skill e pubblica il testo del menu (Markdown) con `SendActivity`
  - Nessun flow del progetto usa più il connettore `shared_teams`: tutte le pubblicazioni su Teams passano ora dall'identità nativa del bot
  - Nuovo flow `ActionTriggerInvioMenu` registrato in `Solution.xml` come RootComponent (type 29), con relativo metadata `.json.data.xml`
- Nota operativa: verificare che `ActionTriggerInvioMenu` sia attivo (ON) in Power Automate dopo l'import (i flow con trigger Recurrence talvolta vengono importati disattivati)

## [1.0.0.67] - 2026-09-08
### Modifiche
- **Fix bug critico**: loop infinito di messaggi su Teams (il flow `ActionPubblicaTabellaOrdini` pubblicava su Teams come identità "Flow bot", separata dal bot; il messaggio veniva ri-letto dal bot come nuovo input utente, ri-innescando `TopicRaccoltaOrdini` all'infinito — confermato riproducibile e confermato risolto disattivando il flow)
  - `ActionPubblicaTabellaOrdini`: rimossa la connessione Teams e l'azione di post diretto; il flow ora restituisce solo il testo della tabella (output `tabella`, formattato in Markdown al posto dell'HTML precedente)
  - Topic `RaccoltaOrdini` e `AnnullaOrdine`: dopo l'invocazione del flow, il testo ricevuto viene pubblicato con `SendActivity` (messaggio nativo del bot, escluso dal re-processing di Bot Framework/Teams)
  - Fix applicato solo ad `ActionPubblicaTabellaOrdini` su richiesta esplicita: gli altri flow che pubblicano su Teams (`ActionChiusuraOrdini`, `ActionInvioMenuTeams`) restano invariati in attesa di conferma che questa soluzione funzioni
  - Da monitorare dopo test in Teams: assenza del loop e corretta pubblicazione della tabella ordini

## [1.0.0.66] - 2026-09-08
### Modifiche
- **Nuova funzionalità**: implementata Fase 3 — Chiusura ordini
  - Nuovo flow `ActionTriggerChiusuraOrdini`: schedulato (Recurrence 18:00 per debug, poi 12:00), blocco weekend, invoca `ExecuteCopilot` con keyword `CHIUDI_ORDINI_GIORNALIERI` (stesso pattern di `ActionTrigger`/`POPOLA_MENU_GIORNALIERO`)
  - Nuovo flow `ActionChiusuraOrdini`: legge `FlacaOrdini` del giorno, aggrega gli ordini per piatto (quantità totale, senza nomi utente), pubblica su Teams la notifica di chiusura (`⏱️ Ordini chiusi. Sto preparando il riepilogo per La Flaca.`) seguita dal riepilogo aggregato
  - Nuovo topic `TopicChiusuraOrdini`: routing tramite `triggerQueries` deterministiche su `CHIUDI_ORDINI_GIORNALIERI` (non instradato via istruzioni GPT, coerente con il pattern dei topic di sistema schedulati)
  - Nessuno stato persistito aggiuntivo: il blocco nuovi ordini/modifiche/annullamenti resta demandato al controllo orario runtime già presente in `RegistraOrdine`/`AnnullaOrdine`
  - Registrati in `Solution.xml` i 2 nuovi flow come RootComponent (type 29), con relativi metadata `.json.data.xml`
- Nota operativa: verificare che il flow `ActionTriggerChiusuraOrdini` sia attivo (ON) in Power Automate dopo l'import, poiché i flow con trigger Recurrence talvolta vengono importati disattivati

## [1.0.0.65] - 2026-09-08
### Modifiche
- **Tentativo di fix (poi corretto)**: disattivato `GenerativeActionsEnabled` in `configuration.json` del bot, ipotizzando un conflitto con `recognizer: GenerativeAIRecognizer` come causa del doppio dispatch dei topic (es. flow `ActionRegistraOrdine` eseguito 2 volte per un solo ordine)
- **Esito negativo**: con `GenerativeActionsEnabled: false` l'agente ha smesso di rispondere alle richieste da Teams — impostazione **ripristinata manualmente a `true`** dall'utente via Maker Portal
- Repository locale risincronizzato con export dal CRM (`pac solution export` + unpack) per allineare `configuration.json`, `bot.xml`, `Solution.xml`, `Customizations.xml` allo stato reale post-modifica manuale
- **Verifica successiva**: dopo il ripristino e la ripubblicazione, i flow partono correttamente **una sola volta** — il doppio dispatch non si è ripresentato. Causa reale quindi non attribuibile al flag `GenerativeActionsEnabled` (che resta necessario a `true`), probabilmente legata a un effetto transitorio del ciclo di import/pubblicazione (cache orchestratore o registrazione canale Teams) verificatosi durante i test intensivi del 03/09. Da tenere monitorato nei prossimi giorni per confermare la stabilità
- Osservazione (non risolta, limite di piattaforma): la tab "Attività" di Copilot Studio non traccia il topic invocato per le sessioni provenienti dal canale Teams quando si usa l'orchestrazione generativa (0 passaggi/turni, stato "Inviata"), a differenza del Test chat interno che traccia sempre correttamente. Non impatta il funzionamento reale (confermato da run history dei flow Power Automate), da monitorare/verificare con Application Insights se necessario

## [1.0.0.64] - 2026-09-03
### Modifiche
- **Fix critico**: istruzioni GPT (`gpt.default`) rese esplicite per impedire risposte generate autonomamente dall'orchestratore che bypassavano i topic
  - Aggiunta direttiva: l'orchestratore non deve mai decidere autonomamente se un piatto è presente nel menu (causa di risposte "a parole" come rifiuto ordine senza invocare `TopicRaccoltaOrdini`, saltando anche la gestione "fuori menu")
  - Aggiunta direttiva: dopo l'esecuzione di un topic, l'orchestratore non deve mai aggiungere un proprio messaggio di commento/conferma — il turno termina con l'output del topic. Mitiga il bug del doppio messaggio di esito ordine (es. "Ordine aggiornato" seguito da "Ordine registrato")

## [1.0.0.63] - 2026-09-03
### Modifiche
- **Fix critico**: risolto doppio dispatch del topic `TopicRaccoltaOrdini` (ogni messaggio, anche un numero valido, attivava due volte la catena `ValidaOrdine → RegistraOrdine → PubblicaTabellaOrdini`, con rischio di righe duplicate in FlacaOrdini)
  - Rimosse le `triggerQueries` (NLU classico) da `TopicRaccoltaOrdini` e `TopicAnnullaOrdine`: il routing resta esclusivamente a carico dell'orchestratore GPT (istruzioni esplicite), eliminando il doppio matching simultaneo NLU + AI orchestrator
- **Fix**: aggiunta guardia in `TopicRaccoltaOrdini` per ignorare messaggi di sola conferma ("si"/"sì"/"no") in ingresso come nuovo turno, evitando la riattivazione indesiderata del topic dopo il click sui pulsanti Sì/No della domanda "piatto fuori menu" (`EndDialog` con `clearTopicQueue`)

## [1.0.0.62] - 2026-09-03
### Modifiche
- **UX**: la domanda di conferma "piatto fuori menu" in `TopicRaccoltaOrdini` ora usa un'Adaptive Card con pulsanti Sì/No (`ClosedListEntity`) invece della risposta testuale libera, evitando che l'utente debba scrivere e "richiamare" l'agente
- Modifica applicata manualmente dall'utente via UI Copilot Studio, sincronizzata dal CRM e verificata (nessun mismatch case-sensitive introdotto)
- Rimossi 2 branch di condizione vuoti (dead code) generati automaticamente dalla UI durante il salvataggio

## [1.0.0.61] - 2026-09-03
### Modifiche
- **Nuova funzionalità**: gestione ordine "fuori menu"
  - `TopicRaccoltaOrdini`: quando il piatto non è trovato nel menu del giorno, invece del rifiuto immediato, l'agente chiede conferma all'utente ("Vuoi ordinarlo comunque come piatto fuori menu?")
  - Se confermato (SI/sì): l'ordine viene registrato con il testo libero digitato dall'utente, gestendo gli stessi esiti del flusso standard (`nuovo`/`aggiornato`/`fuori_orario`) e pubblicando la tabella ordini aggiornata
  - Se non confermato: invariato il messaggio di errore standard
  - `RegistraOrdine`: aggiunta gestione `number = 0` → usa il nome piatto libero (`text_1`) invece del lookup su FlacaMenu per Indice, sia in creazione che in aggiornamento ordine

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
