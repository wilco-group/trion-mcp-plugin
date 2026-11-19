---
name: utenti-ruoli
description: Elenca lo staff e i ruoli di una sede del gestionale Trion, invita persone nuove con il ruolo giusto, revoca o rimanda gli inviti e cambia il ruolo di chi c'è già. Usala quando il Super Admin chiede chi lavora in una sede, quali ruoli ci sono, di invitare o aggiungere un utente, un amministratore, un medico o una segretaria, di revocare un invito o di cambiare un ruolo.
argument-hint: "<slug della sede>"
---

# Utenti e ruoli

Sede: $ARGUMENTS (se vuoto, chiedila).

## Regole

- **L'invito manda subito un messaggio** (email, o WhatsApp se scelto) alla persona. Invita solo chi ti è stato chiesto, con l'email data da chi ti parla: mai indirizzi dedotti o inventati.
- `invite_user` chiede conferma: prima l'anteprima, poi `confirm: true` solo dopo il sì.
- **Clinica in prova.** Se la sede è demo (`get_site`, `isDemo: true`), ricordalo prima di invitare persone vere del cliente.

## Passi

1. `list_users` con `slug`: chi c'è già (nome, email, ruolo base, ruolo personalizzato). Se la persona c'è già, non invitarla di nuovo.
2. `list_roles` con `slug`:
   - ruoli base: `ADMIN` Amministratore, `DOCTOR` Medico, `NURSE`, `SECRETARY`, `VIEWER`;
   - ruoli personalizzati del gruppo, che si usano solo con `advancedRolesActive: true` (modulo `advanced_roles`).
3. Per ogni persona da invitare, raccogli: nome, email, ruolo base, eventuale ruolo personalizzato (`customRoleId`), canale (`EMAIL` predefinito; `WHATSAPP` solo con un cellulare in `phone`), altre sedi del gruppo con lo stesso ruolo (`tenantIds`).
4. `invite_user` senza `confirm`, poi mostra l'anteprima: chi, dove, ruolo, canale. Dopo il sì, lo stesso con `confirm: true`.

## Controlli finali

`list_users` mostra solo lo staff attivo: chi ha appena ricevuto l'invito compare dopo averlo accettato. Per gli inviti aperti usa `call_query` con `name: "users.listInvites"` e `slug` (lo schema con `describe_procedure`). Riporta l'esito per ogni persona.

## Inviti aperti e cambio di ruolo

Non hanno uno strumento dedicato: si fanno con `call_mutation`.

1. **Trova la procedura.** `find_procedure` (es. «inviti», «membro», «ruolo»), poi `describe_procedure` per lo schema. Di solito:
   - `users.revokeInvite` (`inviteId`) revoca un invito aperto;
   - `users.resendInvite` (`inviteId`) lo manda di nuovo, quindi **manda un messaggio**;
   - `users.updateMember` (`userId` più i campi da cambiare) cambia ruolo base, ruolo personalizzato o altri dati di un membro;
   - `customRoles.assignToUser` assegna o toglie un ruolo personalizzato.

   Gli `inviteId` vengono da `call_query` con `users.listInvites`.
2. **Un invito da correggere** (email, ruolo o canale sbagliati) non si modifica: revocalo e invita di nuovo con `invite_user`, ognuno con la sua conferma.
3. **Anteprima.** `call_mutation` con `name`, `input` e `slug`, senza `confirm`. Mostra l'anteprima così com'è: procedura, area, sede e input già validato. Per un cambio di ruolo scrivi anche il ruolo di prima e quello nuovo (da `list_users`).
4. **Conferma.** Dopo un sì esplicito, la stessa chiamata con `confirm: true`. Una persona per volta.

Il ruolo personalizzato vale solo con il modulo `advanced_roles` acceso. Togliere un ruolo personalizzato è sempre possibile.

## Cosa non si fa da qui

- Togliere una persona dallo staff (`users.removeMember`): è escluso da `call_mutation`, si fa dal gestionale (Impostazioni › Utenti).
- Password, verifica in due passaggi e sessioni: esclusi, li gestisce la persona.
