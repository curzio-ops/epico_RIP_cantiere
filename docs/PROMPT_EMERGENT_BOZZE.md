# Prompt per Emergent — bozza e invio delle richieste

Da mandare **dopo** la Consegna B dell'anagrafica clienti (verificata).

```
Richieste cantieri — BOZZA e INVIO (portale + admin). Da fare dopo la Consegna B.

1. Stati della richiesta: "bozza" → "inviata" → (importata, come oggi). Campo `stato_invio` + `inviata_il`,
   `inviata_da`. Migrazione allo startup, idempotente: le richieste esistenti diventano "inviata" (inviata_il =
   creata_il), il loro numero_richiesta resta.
2. Portale — "Nuova richiesta" salva come BOZZA: nessun campo obbligatorio (solo validazione dei formati, es. pezzi
   intero ≥ 1 se compilato), nessun numero_richiesta, nessuna notifica. Pulsanti nel modulo: "Salva bozza" e
   "Invia a Epico".
3. Invio (POST /api/portal/richieste/{id}/invia): validazione completa come oggi (422 per campo), assegna
   numero_richiesta (progressivo attuale), stato inviata, inviata_il/da, registro, push agli admin come oggi.
4. Dopo l'invio, finché non è importata, Brusa può ancora MODIFICARE: validazione completa, registro prima/dopo,
   campo `aggiornata_dopo_invio_il`, push agli admin "Richiesta R26-0001 aggiornata da BRUSA". Pulsante "Ritira in
   bozza" (POST .../ritira): torna bozza, sparisce dalla lista admin, il numero resta assegnato. Eliminazione
   (logica) permessa per bozze e inviate non importate.
5. Importata/collegata: tutto bloccato come oggi (409).
6. Admin "Richieste cantieri": vede SOLO le inviate (mai le bozze); badge = inviate non importate; etichetta
   "Aggiornata il gg.mm.aaaa hh:mm" se modificata dopo l'invio; nel registro si vedono le modifiche. Importa e
   collega rifiutati (409) su bozza.
7. Portale, lista: stato "Bozza" (grigio, etichetta "Bozza — non ancora inviata"), poi i colori attuali.
   Filtro stato con "Bozze". Excel del portale: colonna "Data invio"; le bozze incluse solo se il filtro è "Bozze"
   o "Tutti". Excel admin: mai le bozze.
8. Test: bozza incompleta salvata senza numero e invisibile all'admin; invio con validazione e numero; modifica dopo
   invio con notifica e registro; ritira; import di una bozza → 409; import usa i dati aggiornati; migrazione delle
   esistenti; separazione tra utenti portale invariata.
Save to GitHub (non fare il Redeploy prima della mia verifica).
```
