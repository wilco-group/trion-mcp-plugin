---
name: moduli-integrazioni
description: Accende e spegne moduli e integrazioni di una sede del gestionale Trion (account Audin, WhatsApp, Gmail, Google Calendar, Marketing, multi-sede, ruoli avanzati, prenotazione online, AI) e ne legge lo stato. Usala quando il Super Admin chiede di attivare, disattivare o controllare un modulo, WhatsApp, Audin o un'integrazione.
argument-hint: "<slug della sede>"
---

# Moduli e integrazioni

Sede: $ARGUMENTS (se vuoto, chiedila).

## I moduli

| Chiave | Nome | Cosa fa |
|---|---|---|
| `whatsapp` | Audin (per la clinica «Contatti e moduli online») | Account Audin della sede: contatti dalle campagne, moduli online, copia dei contatti, oblio, telefonate. Acceso, crea l'account del gruppo presso il fornitore. Spento, si ferma tutto ciò che passa da Audin, WhatsApp compreso. |
| `whatsapp_messaging` | WhatsApp | Conversazioni e invii WhatsApp, template, passaggi WhatsApp delle automazioni, promemoria, campagne, credito. Richiede `whatsapp` acceso. |
| `gmail` | Gmail | Email ai pazienti dalla casella Gmail della clinica. Il collegamento lo fa un amministratore della clinica dal gestionale. |
| `google_calendar` | Google Calendar | Appuntamenti copiati nel Google Calendar della clinica. Il collegamento lo fa un amministratore della clinica. |
| `crm_leads` | Marketing | Pipeline commerciale, lead, conversione in pazienti. |
| `multi_sede` | Gestione Multi-Sede | Vista aggregata per chi amministra più sedi. |
| `advanced_roles` | Gestione Avanzata Ruoli | Ruoli personalizzati con permessi a matrice. |
| `online_booking` | Prenotazione Online | Pagina pubblica `/book`: spenta, mostra solo i recapiti. |
| `ai_listening` | Ascolto AI Visita | Registrazione e trascrizione della visita con proposte dei campi. |
| `ai_widgets` | Widget AI dashboard | Widget creati descrivendoli a parole. |
| `app_shop` | Prodotti nell'app | Ordini dall'app con ritiro in clinica. |

Per la clinica Audin non esiste: si parla di «WhatsApp» e di «contatti». Non scrivere «Audin» in testi destinati alla clinica.

## Prima di accendere invii

Prima di accendere un modulo che fa partire messaggi verso persone (`whatsapp_messaging`, `gmail`, `whatsapp` con l'ingresso dei contatti, `online_booking`), controlla con `get_site`:
- `isDemo`: se `true`, dillo; una clinica in prova non dovrebbe scrivere a persone vere;
- `deactivatedAt`: se valorizzato, la sede è disattivata e non si accende nulla.

Chiedi conferma con questi due dati sotto gli occhi.

## Passi

1. `get_site` con `slug`: moduli accesi, `isDemo`, `deactivatedAt`.
2. `integration_health`: stato reale di Audin e WhatsApp, Gmail, Calendar, Fatture in Cloud, Stripe, stampante.
3. **Cambi.** `set_modules` con `modules: [{ key, enabled }]` (più `expiresAt` ISO se il contratto ha una scadenza). L'anteprima arriva quando si spegne un modulo o si accende `whatsapp`, ed elenca gli effetti: mostrala e conferma solo dopo il sì. Senza anteprima, il cambio è già fatto: riassumilo prima e chiedi il sì prima di chiamare.
4. **Ordine.**
   - Per accendere: prima `whatsapp`, poi `whatsapp_messaging`.
   - Per spegnere: prima `whatsapp_messaging`, poi `whatsapp`.
5. **Account Audin che non è partito.** Se `integration_health` dice che l'account manca, usa `ensure_audin_account` (con conferma). Se l'account c'è ma gli eventi no, usa `register_audin_events`.

## Controlli finali

`get_site` e `integration_health` dopo i cambi: riporta moduli accesi e problemi aperti. Gmail e Calendar restano «da collegare» finché la clinica non li collega: non è un errore.
