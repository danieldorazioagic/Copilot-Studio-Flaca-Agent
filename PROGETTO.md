# SL CE - Flaca Agent — Stato Progetto

> Aggiornato: 2026-09-03 | Versione live: **v1.0.0.60**

---

## 📌 Obiettivo

Agente Copilot Studio integrato in Microsoft Teams per automatizzare la raccolta ordini giornalieri del ristorante **La Flaca**.

Canale principale: **gruppo Teams "Flaca"**
Archivio dati: **SharePoint Lists** (FlacaMenu, FlacaOrdini)

---

## 🏗️ Architettura

| Componente | Ruolo |
|---|---|
| **Copilot Studio** | Interfaccia conversazionale (topic + orchestratore AI) |
| **Power Automate** | Logica business, schedulazioni, integrazioni |
| **SharePoint** | Storage giornaliero (FlacaMenu, FlacaOrdini) |
| **Microsoft Teams** | Canale utenti |
| **WhatsApp** | Invio ordine finale al ristorante |

### Flow Power Automate

> ⚠️ **Nota naming**: dal fix manuale via UI Copilot Studio (v1.0.0.59→60), TUTTI i flow risultano rinominati lato CRM con prefisso `Action` (es. "SL CE - Flaca Agent | ActionRegistraOrdine"). I GUID (`flowId`) sono invariati: i topic continuano a invocarli via `flowId` diretto, non tramite le action botcomponent (disattivate — architettura "solo topic").

| Flow | Trigger | Scopo |
|---|---|---|
| `Trigger` | Schedulato 09:00 (lun-ven) | Avvia flusso giornaliero |
| `InvioMenuTeams` | Chiamato da Trigger | Invia menu su Teams |
| `ScriviMenuSuSharePoint` | Chiamato da Trigger | Scrive piatti su FlacaMenu con Indice |
| `ValidaOrdine` | CS Skill (Topic RaccoltaOrdini) | Valida piatto per Indice o Nome |
| `RegistraOrdine` | CS Skill (Topic RaccoltaOrdini) | Crea/aggiorna ordine in FlacaOrdini |
| `AnnullaOrdine` | CS Skill (Topic AnnullaOrdine) | Verifica orario, cerca ed elimina ordine utente per DisplayName |
| `PubblicaTabellaOrdini` | CS Skill (RaccoltaOrdini + AnnullaOrdine, dopo esiti positivi) | Legge tutti gli ordini del giorno e pubblica tabella riepilogativa su Teams |
| `CancellaMenuOggi` | Manuale / schedulato | Reset FlacaMenu + FlacaOrdini |

### SharePoint Lists

**FlacaMenu**
| Colonna | Tipo | Note |
|---|---|---|
| Title | Text | Nome piatto |
| Categoria | Text | Primi, Secondi, ecc. |
| Indice | Number | Progressivo giornaliero |
| DataMenu | Date | Data del menu |
| Attivo | Boolean | Se disponibile |

**FlacaOrdini**
| Colonna | Tipo | Note |
|---|---|---|
| Title | Text | Display name utente |
| Piatto | Text | Nome piatto scelto |
| NumeroPiatto | Number | Indice piatto |
| DataOrdine | Date | Data ordine |
| StatoOrdine | Text | attivo / modificato |
| TimestampModifica | DateTime | Ultima modifica |

---

## ✅ Completato

### Fase 1 — Menu giornaliero
- [x] Trigger schedulato alle 09:00 (lun-ven, blocco weekend)
- [x] `ScriviMenuSuSharePoint`: scrive piatti con Indice progressivo globale
- [x] `InvioMenuTeams`: invia menu formattato nel gruppo Teams
- [x] Blocco sabato/domenica in Trigger e InvioMenuTeams

### Fase 2 — Raccolta ordini
- [x] Topic `RaccoltaOrdini` in Copilot Studio
- [x] `ValidaOrdine`: cerca piatto per Indice o per Nome (senza filtro data)
- [x] `RegistraOrdine`: crea/aggiorna ordine con deduplicazione per utente+data
- [x] Check orario delegato interamente al flow PA (`W. Europe Standard Time` — DST automatico)
- [x] Fix piatto: letto da FlacaMenu via Indice (non da CS AI)
- [x] Fix Titolo: DisplayName via O365 Users connector (non email/UPN)
- [x] Fix binding output `RegistraOrdine`: response body key `text` (pattern CS)
- [x] Fix OData filter: apostrofi nel DisplayName escapati con `replace()`
- [x] Topic: rimosso check orario hardcoded, gestione esito `nuovo`/`aggiornato`/`fuori_orario`
- [x] GPT Instructions: contesto gruppo Teams multi-utente esplicito
- [x] `System.User.PrincipalName` per identificazione utente nel topic
- [x] `System.Activity.Text` pre-popola input ordine (no domanda ridondante)
- [x] **Test v1.0.0.50 su Teams** — superato, flusso stabile

### Fase 2b — Annullamento ordine (v1.0.0.58-60)
- [x] Nuovo Topic `TopicAnnullaOrdine` in Copilot Studio (trigger queries: "annulla il mio ordine", "cancella il mio ordine", ecc.)
- [x] Nuovo Flow `AnnullaOrdine`: verifica orario chiusura, cerca ordine utente per DisplayName, elimina se presente
- [x] Gestione esiti: `annullato` / `non_trovato` / `fuori_orario`
- [x] GPT Instructions aggiornate per il routing dell'intento di annullamento

### Fase 2c — Pubblicazione tabella ordini su Teams (v1.0.0.58-60)
- [x] Nuovo Flow `PubblicaTabellaOrdini`: legge tutti gli ordini del giorno da FlacaOrdini e pubblica tabella (formato testo semplice) nel gruppo Teams
- [x] Invocato da `TopicRaccoltaOrdini` dopo esito `nuovo`/`aggiornato`
- [x] Invocato da `TopicAnnullaOrdine` dopo esito `annullato`

### Fix critico case-sensitivity (v1.0.0.60)
- [x] Diagnosticato e risolto bug introdotto dal re-salvataggio via UI Copilot Studio (necessario per riparare un `InvalidReferenceError` CloudFlow su `TopicAnnullaOrdine`): i nodi `InvokeFlowAction` rigenerati dalla UI scrivevano output in variabili con case diverso (`Topic.nomePiatto`/`Topic.numeroPiatto`/`Topic.esito`) rispetto a quelle usate nel resto dei topic (`Topic.NomePiatto`/`Topic.NumeroPiatto`/`Topic.Esito`), bloccando **tutta** la raccolta ordini (ogni piatto risultava "non trovato")
- [x] Allineati tutti i riferimenti in `TopicRaccoltaOrdini` e `TopicAnnullaOrdine` al case effettivo del binding
- [x] Importato e pubblicato con successo su `AGIC-CopilotStudio_Playground`

### Infrastruttura
- [x] Connection reference SharePoint: `ddo_sharedsharepointonline_31146`
- [x] Connection reference O365 Users: `ddo_shared_office365users`
- [x] Git repository configurato e aggiornato
- [x] ALM: pack/unpack/import con PAC CLI

---

## ⏳ Da fare

### 🔴 Da verificare — Test post fix v1.0.0.60
- [ ] Ritestare su Teams: scrivi nome piatto → `✅ Ordine registrato con successo!` + tabella ordini pubblicata
- [ ] Testare aggiornamento ordine → `✅ Ordine aggiornato con successo!` + tabella aggiornata
- [ ] Testare annullamento ordine → `✅ Ordine annullato con successo!` + tabella aggiornata
- [ ] Testare annullamento senza ordine attivo → `⚠️ Non risulta nessun ordine attivo da annullare per oggi.`
- [ ] Testare fuori orario (ordine e annullamento) → `⚠️ Gli ordini di oggi sono già chiusi.`
- [ ] Verificare DisplayName corretto in SharePoint FlacaOrdini (niente email)

### Fase 3 — Chiusura ordini (18:00 test / 12:00 prod)
- [ ] Flow `ChiusuraOrdini` schedulato alle 12:00
  - Blocca nuovi inserimenti (stato giornata = "chiuso")
  - Legge tutti gli ordini del giorno da FlacaOrdini
  - Aggrega per piatto (count)
  - Notifica Teams: `⏱️ Ordini chiusi. Sto preparando il riepilogo.`
  - Genera tabella riepilogo: Piatto | Quantità

### Fase 4 — Invio ordine al ristorante (WhatsApp)
- [ ] Definire connettore WhatsApp (Business API o alternativa)
- [ ] Flow `InvioWhatsApp`
  - Formatta riepilogo per WhatsApp
  - Invia messaggio al numero La Flaca
  - Gestione errore invio → notifica Teams

### Fase 5 — Conferma ristorante
- [ ] Gestione risposta WhatsApp
- [ ] Se confermato: stato = "confermato" + notifica Teams `✅ Prenotazione effettuata`
- [ ] Se non risponde: mantieni stato "in attesa conferma"

### Miglioramenti pendenti
- [ ] Ripristinare orario chiusura a 12:00 (ora in 18:00 per test)
- [ ] Risposta agente su Teams quando ordine fuori orario (gestita via esito `fuori_orario` ✅)

---

## 🐛 Bug noti / Note tecniche

### Orchestratore CS su Teams
> ⚠️ Su Teams, l'AI orchestratore bypassa i topic e chiama i flow PA come "skills" direttamente.
> **Conseguenza**: tutta la logica business (orario, validazione, dedup) deve stare nei flow PA, non nel topic CS.

### Variabili utente CS in Teams
| Variabile | Risultato in Teams | Note |
|---|---|---|
| `System.User.DisplayName` | Email/UPN | Non usabile |
| `System.User.PrincipalName` | Email/UPN | Usato come identificatore |
| `System.User.FirstName/LastName` | Non testato | Alternativa |
| **O365 Users connector** | DisplayName reale ✅ | Soluzione adottata |

### Pubblicazione dopo import
> Dopo ogni `pac solution import`, il topic CS deve essere **pubblicato manualmente dalla UI di Copilot Studio** perché Teams usi la versione aggiornata.

### ⚠️ Case-sensitivity variabili Power Fx dopo re-salvataggio UI
> Quando Copilot Studio ri-salva un nodo `InvokeFlowAction` tramite la UI Maker Portal (es. per riparare un riferimento `CloudFlow` non trovato), **rigenera il binding di output usando il nome del campo schema esattamente come titolo** (case-sensitive: es. `nomePiatto`, `numeroPiatto`, `esito` — lowercase-first), ignorando eventuali variabili PascalCase (`Topic.NomePiatto`, `Topic.Esito`) usate manualmente nel resto del topic. Questo genera variabili "gemelle" mai riconciliate.
> **Da fare dopo ogni fix manuale in UI**: rileggere l'intero topic (`data` YAML) e verificare che tutti i riferimenti alla stessa variabile logica abbiano case coerente, allineando al case effettivamente prodotto dal binding (non il contrario).
> **Effetto collaterale osservato**: lo stesso salvataggio rinomina anche i Cloud Flow con prefisso `Action` lato Dataverse (non impatta il funzionamento, solo il display name).

---

## 📦 Versioni

| Versione | Commit | Descrizione |
|---|---|---|
| v1.0.0.36 | `6ec8f0f` | Blocco weekend Trigger + InvioMenuTeams |
| v1.0.0.37 | — | Fix ValidaOrdine: rimosso filtro DataMenu |
| v1.0.0.38 | — | Fix Indice in ScriviMenuSuSharePoint (tentativo) |
| v1.0.0.39 | `833d97b` | Fix Indice: cast Float→Int con `int()` ✅ |
| v1.0.0.42 | `38cb6c7` | Fix piatto: letto da FlacaMenu via Indice ✅ |
| v1.0.0.43 | `80da2ef` | Tentativo FirstName+LastName (sostituito) |
| v1.0.0.44 | `8407ea9` | Check orario + O365 Users DisplayName |
| v1.0.0.46 | — | Fix GPT Instructions Teams group + topic trigger queries + PrincipalName |
| v1.0.0.47 | — | Fix binding output RegistraOrdine (rimosso) |
| v1.0.0.48 | — | Orario chiusura spostato a 18:00 per test |
| v1.0.0.49 | — | Check orario delegato solo a PA flow (DST auto) + binding esito ripristinato |
| v1.0.0.50 | — | Fix OData filter apostrofo nel DisplayName ✅ |
| v1.0.0.58 | — | Nuovi flow `AnnullaOrdine` + `PubblicaTabellaOrdini`; nuovo topic `TopicAnnullaOrdine`; `TopicRaccoltaOrdini` invoca pubblicazione tabella dopo esito positivo |
| v1.0.0.59 | — | Tentativo fix `InvalidReferenceError` CloudFlow su `TopicAnnullaOrdine` con action botcomponent (non risolutivo) |
| v1.0.0.60 | — | Fix critico case-sensitivity variabili Power Fx (`nomePiatto`/`numeroPiatto`/`esito`) introdotto dal resave UI CS; sincronizzata struttura locale con CRM (rename flow `Action*`) ✅ |

---

## 🔧 Comandi utili

```powershell
# Pack soluzione
pac solution pack --zipfile "SLCEFlacaAgent_1_0_0_XX.zip" --folder "SLCEFlacaAgent_unpacked" --allowDelete --clobber

# Import soluzione
pac solution import --path "SLCEFlacaAgent_1_0_0_XX.zip" --force-overwrite --publish-changes

# Import con settings (connection references)
pac solution import --path "SLCEFlacaAgent_1_0_0_XX.zip" --force-overwrite --publish-changes --settings-file "import_settings.json"

# Verifica auth
pac auth list

# Lista connessioni ambiente
pac connection list
```

---

*Ultimo aggiornamento: 2026-09-03*
