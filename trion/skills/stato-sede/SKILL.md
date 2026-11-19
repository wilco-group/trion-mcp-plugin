---
name: stato-sede
description: Riepilogo in sola lettura di una sede del gestionale Trion (gruppo, paese, valuta, demo, pubblica, moduli, integrazioni, staff). Usala quando il Super Admin chiede lo stato, la situazione o un controllo di una sede o di una clinica.
argument-hint: "<slug della sede>"
---

# Stato di una sede

Sede: $0. Se manca, chiedila. Se non è uno slug, cercala con `list_sites` e `search`.

Solo letture: questo comando non scrive nulla.

1. `get_site` con `slug`: gruppo, paese, valuta, città, `isDemo`, `isPublic` (vetrina dell'app), `deactivatedAt`, moduli, prenotazione online.
2. `get_currency`: valuta bloccata o no, e il motivo.
3. `integration_health`: problemi aperti per servizio.
4. `list_users`: numero di persone per ruolo base.
5. `list_issuing_companies`: società del gruppo, con il paese.

Rispondi con una scheda breve:
- **Identità:** nome, slug, gruppo, sede principale sì/no, paese, valuta.
- **Stato:** demo, pubblica, disattivata.
- **Moduli accesi**, con la scadenza se c'è.
- **Integrazioni:** solo quelle con un problema, più quelle da collegare.
- **Staff:** conteggi per ruolo.
- **Da fare:** cosa manca per partire (es. prezzi da impostare in una sede in altra valuta, Gmail da collegare, sede ancora demo).
