---
name: cerca-leggi
description: Cerca e legge qualunque dato del gestionale Trion che non ha uno strumento dedicato, passando dalle procedure di sola lettura (find_procedure, describe_procedure, call_query). Usala quando il Super Admin chiede un dato di una sede (appuntamenti, fatture, preventivi, impostazioni, statistiche) che gli altri strumenti non coprono.
argument-hint: "<cosa cercare> [slug della sede]"
---

# Cercare e leggere

Richiesta: $ARGUMENTS

## Passi

1. **Cerca la procedura.** `find_procedure` con una parola chiave in italiano o il nome dell'area (es. «valuta», «fatture elenco», «appuntamenti»). Scegli la procedura di tipo `query` più vicina alla domanda.
2. **Leggi lo schema.** `describe_procedure` con `name`: dice se è una query e dà lo schema JSON dell'ingresso. Costruisci l'`input` solo con i campi dello schema.
3. **Esegui.** `call_query` con:
   - `name`;
   - `input`;
   - `slug` o `tenantId` per i dati di una sede: gira come amministratore di quella sede;
   - senza sede: solo le procedure `admin.*` della piattaforma.
4. Rispondi con i dati che servono, in tabella se sono righe. Non incollare risposte grezze lunghe.

## Limiti

- **Solo letture.** Con una mutation `call_query` rifiuta. Per scrivere usa prima lo strumento dedicato, se c'è; altrimenti `call_mutation` con la stessa ricerca (`find_procedure` → `describe_procedure`). `call_mutation` risponde sempre prima con l'anteprima: mostrala e richiama con `confirm: true` solo dopo un sì esplicito. Esclude sicurezza dell'account, accesso MCP, link e firme del paziente, cancellazioni definitive e oblio, chiavi dei pagamenti, invii massivi e segreti: per quelle si va dal gestionale.
- **Procedure escluse:** verifica in due passaggi, cancellazione dell'account, link pubblici e token MCP.
- **Dati dei pazienti.** Leggili solo se servono alla domanda e non copiarli in file, commit o messaggi fuori dalla conversazione.
- **Liste lunghe.** Usa i filtri e la paginazione dello schema invece di scaricare tutto.
