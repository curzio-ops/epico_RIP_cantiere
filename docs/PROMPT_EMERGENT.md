# Prompt per Emergent (AGENDA)

Copia tutto il blocco qui sotto in Emergent. La specifica completa è in `docs/SPECIFICA.md`: se Emergent ha dubbi,
incolla anche quella.

---

```
NUOVA FUNZIONE: "Richieste cantieri" nel Portale BRUSA (+ pagina admin). Leggi tutto prima di iniziare.

CONTESTO
Il Cantiere Nautico Brusa ci assegna molte riparazioni di selleria. Oggi il portale /portale è in sola lettura.
Vogliamo che Brusa possa INSERIRE le richieste, che noi le importiamo come lavoro con 1 clic, e che la loro lista si
aggiorni da sola. Deve funzionare per più cantieri: legalo all'utente portale, mai scrivere "Brusa" nel codice.

REGOLE
- Campi della richiesta = STESSI NOMI dei `fields` di mongo_lavori (nessun mapping all'import).
- Numero lavoro, presa a carico, riconsegna, stato: NON copiarli nella richiesta; leggili dal lavoro collegato ad
  ogni GET (un solo dato vero).
- Importi mai float: stringhe decimali in Mongo, Decimal nei calcoli, a video "CHF 1'290.00".
- Date ISO in DB, "gg.mm.aaaa" a video e in Excel. Fuso Europe/Zurich.
- Separazione: un utente portale vede SOLO le sue richieste. Il portale mostra SOLO preventivi/offerte, mai fatture,
  solleciti o altri allegati.
- Eliminazione solo logica. Registro su ogni richiesta: chi, quando, azione, valori prima/dopo.
- Non toccare il comportamento esistente del portale (Riassunto, Tutti i lavori, Agenda mensile, foto).

1) MODELLO DATI
Nuova collection `cantiere_richieste`:
{ _id: uuid, portal_username, numero_richiesta: "R26-0001" (progressivo per anno e utente, in mongo_counters),
  data_registrazione: "YYYY-MM-DD" (oggi, messa dal server),
  fields: {
    "Natante / Veicolo / Macchinario (Tipo) / Oggetto": str  (obbl.)  etichetta "Marca e modello natante"
    "Numero di targa": str (maiuscolo)                               etichetta "Numero di targa"
    "Cognome": str (obbl.)                                           etichetta "Cognome cliente finale"
    "Nome": str                                                      etichetta "Nome cliente finale"
    "Luogo di lavorazione": str (obbl., dalla lista)                 etichetta "Luogo di stazionamento"
    "Luogo di Stazionamento": str (obbl. se scelto Altro)            etichetta "Ormeggio"
    "Parte da lavorare": str (obbl.)                                 etichetta "Materiale da lavorare"
    "Tipo di lavorazione": str (obbl., testo lungo)                  etichetta "Descrizione lavorazione"
    "Numero di pezzi ritirati": int >= 1 (obbl.)                     etichetta "Numero di pezzi"
    "Note": str (testo lungo)                                        etichetta "Osservazioni"
  },
  lavoro_uid: null, job_code: null, importata_il: null, importata_da: null,
  creata_il, creata_da, modificata_il, eliminata: false, registro: [] }
Indici: (portal_username, eliminata, creata_il), job_code.

portal_users: aggiungi `cliente_ninox_id` (int, cliente di fatturazione del cantiere, es. Cantiere Nautico Brusa) e
`richieste_abilitate` (bool, default false). Aggiungili al form in Impostazioni > Utenti portale (con ricerca cliente).
Per l'utente `brusa`: richieste_abilitate = true e cliente = "Cantiere Nautico Brusa".

2) LISTA PORTI
Stessa lista del campo "Luogo di lavorazione" (field_options.resolve_options). Nel modulo portale mostra l'ultima voce
"(altro) scrivere nel stazionamento" come "Altro": se scelta, "Ormeggio" diventa obbligatorio. Salva il valore
originale della lista (non "Altro"), così il lavoro AGENDA resta coerente.

3) API PORTALE (JWT portale; filtro sempre per portal_username del token)
GET    /api/portal/richieste?anno=&stato=&q=     lista con campi di ritorno e preventivo risolti (vedi 5)
GET    /api/portal/richieste/form-config         lista porti
POST   /api/portal/richieste                     crea (403 se richieste_abilitate è false)
PUT    /api/portal/richieste/{id}                solo se non importata, altrimenti 409
DELETE /api/portal/richieste/{id}                logica, solo se non importata
GET    /api/portal/richieste/export.xlsx?anno=&stato=&q=
GET    /api/portal/richieste/{id}/preventivo/{nome_file}   link temporaneo Dropbox (dropbox_allegati.temp_link);
       404 se l'allegato non è documento_erufatt con erufatt.tipo in (preventivo, offerta) del lavoro collegato a
       una richiesta di quell'utente.
stato = da_prendere | in_lavorazione | riconsegnato | annullato (calcolato, vedi 6).

4) PAGINA ADMIN "Richieste cantieri"
Sidebar, sezione "Portali Clienti", sotto "Portale BRUSA", con badge = richieste non importate.
Tabella con filtro per cantiere, "solo nuove", ricerca. Per ogni riga:
- "Apri lavoro": crea il lavoro con mongo_repo.create_lavoro_mongo_first copiando TUTTI i `fields` della richiesta,
  più: "Clienti": [cliente_ninox_id dell'utente portale], "Stato del lavoro": "Registrato",
  "Data ora Check-in": adesso (Europe/Zurich, stesso formato del check-in mobile), "Ricevente": admin che importa.
  Poi salva lavoro_uid, job_code, importata_il, importata_da. Atomico: find_one_and_update con lavoro_uid null
  (doppio clic = nessun doppione, risposta 409 "già importata").
  Dopo l'import apri la scheda del lavoro creato.
- "Collega a lavoro esistente" (body {job_code}): per lavori già aperti a mano, verifica che il lavoro esista.
- "Scollega" (conferma + registro): corregge un collegamento sbagliato.
- "Scarica Excel" per cantiere.
API: GET /api/admin/richieste-cantieri, POST .../{id}/importa, POST .../{id}/collega, POST .../{id}/scollega,
GET /api/admin/richieste-cantieri/export.xlsx?portal_username=
Notifica push agli admin a ogni nuova richiesta (riusa push_service).

5) CAMPI DI RITORNO (letti dal lavoro collegato, a ogni GET, in batch, niente N+1)
- NUMERO DI LAVORO = numero_lavoro / job_code (es. L26182)
- DATA PRESA A CARICO EPICO = fields "Data ora Check-in"
- DATA RICONSEGNA A CANTIERE = fields "Data / Ora (Check-Out)"
- Stato = fields "Stato del lavoro"
- Preventivo: da mongo_allegati del lavoro, kind documento_erufatt con erufatt.tipo in (preventivo, offerta);
  prendi il più recente per erufatt.data_documento; restituisci data_documento, numero, imponibile, totale, valuta,
  nome_file e il conteggio degli altri ("+n", elencabili).
Lavoro cancellato (tombstone) o "Annullato / Rifiutato" → stato annullato.

6) COLORI RIGA (portale e admin)
- non importata → ambra chiaro, etichetta "Da prendere a carico"
- importata senza Check-Out → nessun colore, etichetta = Stato del lavoro
- "Data / Ora (Check-Out)" valorizzata → VERDE, etichetta "Riconsegnato"
- annullato → grigio barrato

7) PORTALE: nuova scheda "Richieste" (prima scheda, stile attuale del portale, ottimizzata per telefono)
- Pulsante "Nuova richiesta" → modulo con i campi del punto 1 raggruppati con un titolino, in quest'ordine:
  Natante (marca e modello, targa) → Cliente finale (cognome, nome) → Ubicazione (luogo di stazionamento, ormeggio)
  → Lavoro (materiale, descrizione, pezzi, osservazioni). Data di registrazione mostrata, non modificabile.
- Tabella: Data reg. | N. richiesta | Natante | Targa | Cliente finale | Luogo | Ormeggio | Materiale | Descrizione |
  Pezzi | Osservazioni | N. lavoro | Presa a carico | Riconsegna | Stato | Data prev. | N. prev. | Importo IVA escl. |
  Totale IVA incl. | PDF.  Su telefono: card con le stesse informazioni.
- Filtri: anno, stato, ricerca testo. Refresh automatico ogni 2 minuti come le altre schede.
- Modifica/elimina solo sulle righe non importate.
- Pulsante "Scarica Excel".

8) EXCEL (openpyxl, già installato)
Nome "Richieste_<display_name>_AAAA-MM-GG.xlsx". Colonne come la tabella del punto 7 (PDF escluso), intestazione in
grassetto, riga d'intestazione bloccata, filtro automatico, larghezze adatte. Date gg.mm.aaaa (celle data vere).
Importi come numeri con formato #'##0.00. Righe riconsegnate con fondo verde C6EFCE, annullate grigio. Rispetta i
filtri attivi.

9) ENDPOINT ESTERNO EruFATT (modifica)
POST /api/external/erufatt/lavori/{codice}/allegati: accetta questi campi Form OPZIONALI (default ""):
tipo, numero, data_documento (YYYY-MM-DD), imponibile, totale, valuta.
Se `tipo` è presente, salva sul riferimento in mongo_allegati:
erufatt: {tipo, numero, data_documento, imponibile, totale, valuta}  (importi come stringa, validati come decimali;
se non validi → 422). Anche nel caso "già presente" (stesso nome e sha256) aggiorna `erufatt` se mancante.
EruFATT manda già questi campi: nessuna nuova variabile d'ambiente.

10) TEST (pytest, obbligatori)
- creazione con validazione (obbligatori, Altro → ormeggio obbligatorio, pezzi >= 1)
- utente portale A non vede/modifica/scarica richieste di B (lista, PUT, DELETE, Excel, preventivo → 404)
- modifica/elimina bloccate dopo import (409)
- import crea il lavoro con tutti i fields, Clienti, Stato Registrato, Check-in; doppio import → 409, un solo lavoro
- collega/scollega con registro
- campi di ritorno letti dal lavoro (cambio Check-Out → stato riconsegnato)
- preventivo: offerta visibile, fattura dello stesso lavoro NON visibile
- Excel: colonne, date, importi numerici, riga verde
- endpoint EruFATT: campi erufatt salvati; importo non decimale → 422; senza campi → comportamento attuale invariato

Alla fine: elenca i file toccati, i test eseguiti con esito, e ricordami Save to GitHub + Redeploy su Coolify.
```
