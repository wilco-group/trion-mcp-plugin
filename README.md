# Trion gestionale — plugin per Claude Code

Collega Claude Code al server MCP del gestionale Trion (`https://gestionale.trionlab.it/api/mcp`) e aggiunge le procedure guidate del team Trion: clinica nuova, sede estera, listino e prezzi, utenti e ruoli, moduli e integrazioni, stato di una sede, ricerca dei dati.

Serve un account del **team Trion** (Super Admin) con la **verifica in due passaggi attiva**. Il plugin non contiene credenziali: senza account il server risponde 401.

## Installazione

```
claude plugin marketplace add wilco-group/trion-mcp-plugin
claude plugin install trion@trion-gestionale
```

Poi, in Claude Code:

1. `/mcp` → `trion-gestionale` → **Authenticate**.
2. Si apre il browser sul gestionale: entra col tuo account e il codice della verifica in due passaggi, controlla il nome del client e premi **Consenti**.
3. Prova con `/trion:stato-sede <slug della sede>` o chiedi a parole, ad esempio «quali sedi ci sono nel gestionale?».

Se una cartella di progetto ha un suo `.mcp.json` con un server `trion-gestionale`, vale quello: apri Claude Code fuori da quella cartella.

## Procedure

| Comando | Cosa fa |
|---|---|
| `/trion:stato-sede` | Riepilogo in sola lettura di una sede |
| `/trion:nuova-clinica` | Apre un gruppo con le sue sedi dai dati del contratto |
| `/trion:sede-estera` | Paese, valuta e prezzi di una sede fuori dall'Italia |
| `/trion:listino-prezzi` | Prezzi di sede del listino e delle tipologie |
| `/trion:utenti-ruoli` | Staff, ruoli e inviti di una sede |
| `/trion:moduli-integrazioni` | Moduli e integrazioni di una sede |
| `/trion:cerca-leggi` | Lettura di qualunque dato tramite le procedure di sola lettura |

Ogni scrittura mostra prima un'anteprima e parte solo dopo la conferma; tutto finisce nel registro del gestionale con il tuo nome.

## Aggiornare

```
claude plugin marketplace update trion-gestionale
```
