---
name: nuova-clinica
description: Apre nel gestionale Trion una clinica nuova (gruppo con le sue sedi, anche estere) dai dati del contratto, con moduli, società emittenti e account Audin, lasciandola demo e non pubblica. Usala quando il Super Admin chiede di «aprire», «creare» o «attivare» un cliente o una clinica nuova (es. un gruppo con una sede in Italia e una in Svizzera).
argument-hint: "[nome del cliente]"
---

# Aprire una clinica nuova

Cliente: $ARGUMENTS (se vuoto, chiedilo).

Lavori sul server MCP `trion-gestionale` con il token del Super Admin. Ogni scrittura finisce nel registro del gestionale col nome del token.

## Regole

- **Non inventare dati.** Ragione sociale, partite IVA, indirizzi, paese, valuta e moduli vengono dai contratti o da chi ti parla. Se un dato manca, chiedilo; se non è necessario per aprire, lascialo vuoto e mettilo nel riepilogo finale.
- **Strumenti con conferma** (`create_group_with_sites`, `set_modules`, `ensure_audin_account`, `invite_user`, `call_mutation`). Chiamali prima senza `confirm` e mostra l'anteprima così com'è, `problems` compresi. Poi richiamali con gli stessi dati più `confirm: true`, solo dopo un sì esplicito.
- **Strumenti che scrivono senza anteprima** (`update_site`, `create_issuing_company`, `set_currency`, prezzi). Prima di chiamarli riassumi cosa scriverai e aspetta il sì.
- **Il server non ricorda la sede.** Passa `slug` o `tenantId` a ogni strumento di sede.
- **Niente utenti né pazienti** se il cliente non lo chiede esplicitamente.
- **La clinica resta demo e non pubblica** finché non parte davvero.

## Passi

1. **Dati dal contratto.** Raccogli e mostra in tabella, per conferma:
   - nome del gruppo (di solito il marchio del cliente);
   - per ogni sede: nome, città, indirizzo, CAP, provincia (solo Italia), telefono, partita IVA o codice fiscale, paese (`IT` o `CH`), valuta (`EUR` o `CHF`);
   - quale sede è la principale: è la sede di riferimento del gruppo e i prezzi del gruppo sono nella sua valuta;
   - moduli compresi nel contratto (vedi la skill `moduli-integrazioni` per il significato di ognuno);
   - società che fatturano (ragione sociale, partita IVA, sezionale, IBAN).
2. **Doppioni.** `list_sites` con `search` sul nome e sulla città di ogni sede. Se il cliente esiste già, fermati e dillo.
3. **Gruppo e sedi.** `create_group_with_sites` con:
   - `groupName`;
   - `sites` (paese e valuta per le sedi estere, es. `{ name: "Clinica Esempio Lugano", city: "Lugano", country: "CH", currency: "CHF", zip: "6900" }`);
   - `primaryIndex` = indice della sede principale;
   - `isDemo: true`;
   - `isPublic: false`: le sedi restano fuori dalla vetrina dell'app finché non partono.

   Mostra l'anteprima: slug, paese, valuta, sede principale, `problems`. Con uno slug già in uso, proponi uno `slug` diverso e rifai l'anteprima. Poi conferma.
4. **Non pubblica.** Controlla `isPublic` con `get_site` su ogni sede. Se una sede risulta pubblica, dopo il sì usa `update_site` con `slug` e `isPublic: false`.
5. **Moduli.** Per ogni sede `set_modules` con i moduli del contratto (`[{ key, enabled: true }]`):
   - Prima l'account Audin (`whatsapp`), poi `whatsapp_messaging`: senza l'account WhatsApp non si accende.
   - Accendere `whatsapp` crea l'account Audin del gruppo presso il fornitore, una volta per tutto il gruppo.
   - Mostra anteprima ed effetti, poi conferma.
6. **Account Audin** (solo se il contratto lo comprende):
   - `integration_health` sulla sede principale;
   - se l'account non c'è, `ensure_audin_account` (anteprima, conferma);
   - poi `register_audin_events` per registrare i webhook.
7. **Società emittenti.**
   - `list_issuing_companies` sulla sede principale: sono del gruppo.
   - Per ogni società mancante, `create_issuing_company` dopo il sì, con `slug` della sede principale e: `country`, `name`, `vatNumber`, `seriesPrefix` (predefinito `FAT`), `iban`, più indirizzo, città e CAP.
   - **Società italiana:** `country` vuoto o `IT`, partita IVA di 11 cifre, `regimeFiscale` (predefinito `RF01`), provincia, PEC e codice SDI se li hai.
   - **Società svizzera:** `country: "CH"`, numero IVA `CHE-123.456.788`, con o senza punti e con il suffisso IVA, MWST o TVA facoltativo; il gestionale lo salva normalizzato. NPA di 4 cifre. Niente regime fiscale, provincia, PEC né codice SDI.
   - Il numero IVA svizzero ha una cifra di controllo: se viene rifiutato, ricontrollalo sul contratto o sul registro IDI. Non correggerlo a caso.
   - Una società svizzera fattura solo in PDF: niente fattura elettronica, Fatture in Cloud né Sistema TS.
8. **Sedi in un'altra valuta.** Per una sede con valuta diversa dalla principale, i prezzi del gruppo non valgono: segui la skill `sede-estera`, poi `listino-prezzi`.
9. **Utenti.** Solo se il cliente li ha chiesti: skill `utenti-ruoli`.

## Controlli finali

Per ogni sede:
- `get_site`: gruppo, paese, valuta, `isDemo: true`, `isPublic: false`, moduli accesi come da contratto.
- `integration_health`: stato di Audin e WhatsApp; Gmail e Calendar restano «da collegare», li collega la clinica.

Chiudi con un riepilogo in tabella (sede, slug, paese, valuta, principale, moduli, società con il paese) e l'elenco di cosa resta da fare:
- dati mancanti;
- prezzi di sede da impostare;
- collegamenti Gmail e Calendar che deve fare la clinica;
- messa in produzione con `update_site`: `isDemo: false` e `isPublic: true`, solo quando il Super Admin lo chiede.
