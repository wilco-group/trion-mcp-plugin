---
name: listino-prezzi
description: Legge e imposta i prezzi di sede del listino e delle tipologie di appuntamento nel gestionale Trion, trovando le voci «prezzo da impostare»; cambia anche il prezzo del gruppo e aggiunge voci nuove al listino. Usala quando il Super Admin chiede dei prezzi di una sede o del gruppo, del listino, delle tipologie, dei prezzi in franchi o di una voce nuova.
argument-hint: "<slug della sede>"
---

# Listino e prezzi di sede

Sede: $ARGUMENTS (se vuoto, chiedila).

## La regola

Listino e tipologie sono del gruppo. Il prezzo valido in una sede è:

1. il prezzo di sede, se c'è;
2. altrimenti il prezzo del gruppo, ma solo se la sede ha la stessa valuta della sede principale;
3. altrimenti nessun prezzo: è «prezzo da impostare».

## Passi

1. `get_currency` della sede: valuta della sede.
2. **Listino.** `list_catalog` con `slug` (`limit: 100`, più pagine se servono). Per ogni voce:
   - `price` = prezzo del gruppo;
   - `sitePrice` = prezzo di sede o `null`;
   - `sitePriceEffective` = prezzo valido in sede, `null` se da impostare.
3. **Tipologie.** `list_appointment_types` con `slug`: per ogni tipologia, prezzo della sede e `priceMissing` (`true` = da impostare).
4. Mostra due tabelle (listino e tipologie), con le voci da impostare in cima.
5. **Prezzi.** Chiedi i prezzi a chi ti parla, nella valuta della sede: non convertire e non inventare. Riassumi le modifiche in una tabella (voce, prezzo attuale, prezzo nuovo) e aspetta il sì. Poi, voce per voce:
   - listino: `set_catalog_site_price` con `id` e `price`; `price: null` toglie il prezzo di sede;
   - tipologie: `set_appointment_type_site_price` con `id` e `price`; `price: null` torna al prezzo del gruppo.

## Prezzo del gruppo e voci nuove

Non hanno uno strumento dedicato: si fanno con `call_mutation`.

1. **Trova la procedura.** `find_procedure` (es. «listino», «tipologie»), poi `describe_procedure` per lo schema dell'ingresso. Di solito:
   - `salesCatalog.create` per una voce nuova del listino;
   - `salesCatalog.update` per il prezzo del gruppo di una voce;
   - `appointmentTypes.update` per il prezzo del gruppo di una tipologia.

   Costruisci l'`input` solo con i campi dello schema.
2. **Attenzione al prezzo del gruppo.** È nella valuta della sede principale e vale in tutte le sedi del gruppo con quella valuta che non hanno un prezzo di sede. Prima di cambiarlo, dillo a chi ti parla, con l'elenco delle sedi toccate (`list_groups`).
3. **Anteprima.** `call_mutation` con `name`, `input` e `slug` della sede, senza `confirm`. Mostra l'anteprima così com'è: procedura, area, sede e input già validato. Un input sbagliato dà l'errore già qui, campo per campo: correggilo e rifai l'anteprima.
4. **Conferma.** Dopo un sì esplicito, la stessa chiamata con `confirm: true`. Una voce per volta, ognuna con la sua anteprima.

Una voce nuova nasce col prezzo del gruppo. In una sede con un'altra valuta resta «da impostare»: poi `set_catalog_site_price`. Per il prezzo di sede usa sempre gli strumenti dedicati: `call_mutation` li rifiuta e rimanda a loro.

## Controlli finali

Rileggi `list_catalog` e `list_appointment_types`, poi riporta:
- quante voci hanno cambiato prezzo;
- quante restano da impostare;
- eventuali errori, voce per voce.

## Cosa non si fa da qui

Eliminare voci del listino o tipologie: è una cancellazione, si fa dal gestionale (Impostazioni › Listino, Tipologie).
