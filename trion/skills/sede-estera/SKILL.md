---
name: sede-estera
description: Configura nel gestionale Trion una sede fuori dall'Italia (oggi la Svizzera), con paese, valuta e prezzi di sede, e spiega cosa si spegne da solo. Usala quando il Super Admin parla di una sede estera, svizzera, in franchi o in CHF, o di cambiare paese o valuta a una sede.
argument-hint: "<slug della sede>"
---

# Sede fuori dall'Italia

Sede: $ARGUMENTS (se vuoto, chiedila e trovala con `list_sites`).

## Cosa sapere prima

- **Paesi:** oggi `IT` e `CH`. Vuoto vale Italia.
- **Valute:** `EUR` e `CHF`. Paese e valuta sono indipendenti: cambiare paese non cambia la valuta.
- **Cosa si spegne da solo** fuori dall'Italia, senza cancellare dati:
  - fatturazione elettronica (SDI, XML FatturaPA, Fatture in Cloud);
  - Sistema TS;
  - stampante fiscale RCH con scontrini e chiusura Z;
  - diciture IVA e imposta di bollo sul PDF;
  - codice fiscale obbligatorio del paziente;
  - controlli di P.IVA a 11 cifre, CAP a 5 cifre e provincia.

  Il CAP svizzero ha 4 cifre e il telefono prende il prefisso +41.
- **Blocco della valuta:** si cambia solo finché la sede non ha incassi, fatture o preventivi. Dopo, `get_currency` dà `locked: true` con il motivo, e il cambio non si può più fare: i documenti restano nella loro valuta.
- **Prezzi:**
  - i prezzi del gruppo sono nella valuta della sede principale;
  - in una sede con valuta diversa ogni voce di listino e ogni tipologia senza prezzo di sede è «prezzo da impostare»: non si offre in preventivi e fatture, e la visita nasce senza importo;
  - nessuna conversione automatica, mai.

## Passi

1. `get_site` e `get_currency` della sede: paese, valuta, `locked`, `lockReason`.
2. **Paese.** Se è da cambiare, dopo il sì `update_site` con `slug` e `country` (es. `"CH"`), più CAP e indirizzo corretti se servono.
3. **Valuta.** Se è da cambiare e `locked` è `false`, dopo il sì `set_currency` con `currency`. Se è `locked`, fermati, riporta `lockReason` e non cercare strade alternative.
4. **Prezzi di sede.** Segui la skill `listino-prezzi`: elenca le voci da impostare e chiedi i prezzi nella valuta della sede. Non convertire da euro di tua iniziativa.
5. **Società emittente.** Se la sede fattura con una società del paese, controlla con `list_issuing_companies` che ci sia e che abbia il paese giusto. Se manca, dopo il sì usa `create_issuing_company` con:
   - `country: "CH"`;
   - numero IVA `CHE-123.456.788` (suffisso IVA, MWST o TVA facoltativo; c'è una cifra di controllo);
   - NPA di 4 cifre;
   - niente regime fiscale né provincia.

   Una società svizzera fattura solo in PDF, senza diciture italiane.
   Una società già creata senza paese vale come italiana. Per cambiarle il paese:
   - leggi la società con `list_issuing_companies`;
   - cerca la procedura con `find_procedure` («società emittenti») e leggi lo schema con `describe_procedure`: di solito è `issuingCompanies.update`;
   - l'`input` vuole la società intera, non solo i campi che cambiano. Riporta `id` e i dati che restano uguali, più `country: "CH"`, numero IVA svizzero e NPA di 4 cifre, senza provincia, PEC, codice SDI né regime fiscale;
   - `call_mutation` con `name`, `input` e `slug` della sede principale, senza `confirm`. Mostra l'anteprima: un numero IVA che non va dà l'errore già qui. Poi, dopo il sì, la stessa chiamata con `confirm: true`.

   Da lì le fatture nuove di quella società vanno solo in PDF; quelle già emesse restano com'erano.

## Controlli finali

- `get_site` e `get_currency`: paese e valuta giusti.
- Conteggio delle voci di listino e delle tipologie ancora «da impostare».
- Riepilogo di cosa si è spento per il paese e di cosa resta da fare.
