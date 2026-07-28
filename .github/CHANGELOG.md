# CHANGELOG — SL CE - Flaca Agent

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
