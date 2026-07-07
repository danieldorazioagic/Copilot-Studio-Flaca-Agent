# SL CE - Flaca Agent — Stato Progetto

> Aggiornato: 2026-07-07 | Versione live: **v1.0.0.44**

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

| Flow | Trigger | Scopo |
|---|---|---|
| `Trigger` | Schedulato 09:00 (lun-ven) | Avvia flusso giornaliero |
| `InvioMenuTeams` | Chiamato da Trigger | Invia menu su Teams |
| `ScriviMenuSuSharePoint` | Chiamato da Trigger | Scrive piatti su FlacaMenu con Indice |
| `ValidaOrdine` | CS Skill | Valida piatto per Indice o Nome |
| `RegistraOrdine` | CS Skill | Crea/aggiorna ordine in FlacaOrdini |
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
- [x] Check orario 10:00–12:00 nel flow PA (indipendente dall'orchestratore CS)
- [x] Fix piatto: letto da FlacaMenu via Indice (non da CS AI)
- [x] Fix Titolo: DisplayName via O365 Users connector (non email/UPN)

### Infrastruttura
- [x] Connection reference SharePoint: `ddo_sharedsharepointonline_31146`
- [x] Connection reference O365 Users: `ddo_shared_office365users`
- [x] Git repository configurato e aggiornato
- [x] ALM: pack/unpack/import con PAC CLI

---

## ⏳ Da fare

### Fase 3 — Chiusura ordini (12:00)
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
- [ ] Testare v1.0.0.44 su Teams: Titolo = DisplayName, blocco orario
- [ ] Risposta agente su Teams quando ordine fuori orario (`esito = "fuori_orario"`)
- [ ] Gestione risposta agente dopo RegistraOrdine (messaggio conferma/errore a utente)

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
| v1.0.0.44 | `8407ea9` | Check orario + O365 Users DisplayName ⏳ test |

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

*Ultimo aggiornamento: 2026-07-07*
