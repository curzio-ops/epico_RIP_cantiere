# Richieste di riparazione dai cantieri — Specifica

Versione 1 — 30.09.2026. Primo cantiere: **Cantiere Nautico Brusa** (utente portale `brusa`).
Il modulo è pensato per più cantieri (Hammer, Kohler…): tutto è legato all'utente portale, niente è scritto "Brusa" nel codice.

## 1. Obiettivo

Durante l'inverno Brusa assegna molte riparazioni di selleria, oggi annotate in modo disordinato. Con questo modulo:

1. **Brusa** inserisce ogni lavoro da fare in un modulo del suo portale e vede la lista completa con lo stato.
2. **Brusa** scarica la lista in Excel quando vuole.
3. **Epico**, con un clic, apre il lavoro in AGENDA con i campi già compilati.
4. Da quel momento la riga di Brusa mostra da sola numero lavoro, data di presa a carico, data di riconsegna e, se
   esiste, il preventivo EruFATT (data, numero, importo, PDF).
5. A riconsegna avvenuta la riga diventa **verde**.

## 2. Campi della richiesta

I nomi tecnici sono **identici** ai `fields` dei lavori AGENDA (`mongo_lavori.fields`): l'import copia i valori così come sono,
senza mapping.

| # | Etichetta nel modulo Brusa | Chiave (= campo lavoro AGENDA) | Tipo | Obbl. | Note |
|---|---|---|---|---|---|
| **Registrazione** |||||
| 1 | Data di registrazione | `data_registrazione` (solo richiesta) | data | auto | Oggi, messa dal server, non modificabile |
| **Natante** |||||
| 2 | Marca e modello natante | `Natante / Veicolo / Macchinario (Tipo) / Oggetto` | testo | sì | Es. "Pardo 43" |
| 3 | Numero di targa | `Numero di targa` | testo | no | Maiuscolo, es. "TI 1234" |
| **Cliente finale** |||||
| 4 | Cognome cliente finale | `Cognome` | testo | sì | Proprietario del natante (il cliente del lavoro, `Clienti`, resta il cantiere) |
| 5 | Nome cliente finale | `Nome` | testo | no | |
| **Ubicazione** |||||
| 6 | Luogo di stazionamento | `Luogo di lavorazione` | scelta | sì | Lista porti dell'AGENDA (vedi §3) + "Altro" |
| 7 | Ormeggio | `Luogo di Stazionamento` | testo | sì se "Altro" | Posto barca / pontile; con "Altro" va scritto qui il luogo |
| **Lavoro** |||||
| 8 | Materiale da lavorare | `Parte da lavorare` | testo | sì | Es. "Tendalino", "Cuscineria pozzetto" |
| 9 | Descrizione lavorazione | `Tipo di lavorazione` | testo lungo | sì | |
| 10 | Numero di pezzi | `Numero di pezzi ritirati` | intero ≥ 1 | sì | |
| 11 | Osservazioni | `Note` | testo lungo | no | |
| **Ritorno da Epico** (sola lettura per Brusa, letti dal lavoro collegato) |||||
| 12 | NUMERO DI LAVORO | `numero_lavoro` del lavoro (es. `L26182`) | — | — | |
| 13 | DATA PRESA A CARICO EPICO | `Data ora Check-in` | — | — | |
| 14 | DATA RICONSEGNA A CANTIERE | `Data / Ora (Check-Out)` | — | — | Se valorizzata → riga verde |
| 15 | Stato | `Stato del lavoro` | — | — | Utile a Brusa, nessuna modifica |
| **Preventivo** (da EruFATT, sola lettura) |||||
| 16 | Data preventivo | allegato EruFATT `data_documento` | — | — | Il più recente di tipo preventivo/offerta |
| 17 | Numero preventivo | allegato EruFATT `numero` | — | — | |
| 18 | Importo IVA escl. | allegato EruFATT `imponibile` | — | — | `CHF 1'290.00` |
| 19 | Totale IVA incl. | allegato EruFATT `totale` | — | — | |
| 20 | PDF preventivo | allegato in `mongo_allegati` | — | — | Apertura dal portale |

Nota sui nomi: in AGENDA la **lista dei porti** si chiama `Luogo di lavorazione` e l'**ormeggio** si chiama
`Luogo di Stazionamento`. Nel modulo Brusa le etichette restano quelle che usa il cantiere (Luogo di stazionamento /
Ormeggio); i dati finiscono nei campi AGENDA corrispondenti.

## 3. Lista porti

La stessa lista del campo `Luogo di lavorazione` dell'AGENDA, gestita in Impostazioni → Liste & Opzioni
(`field_options.resolve_options`). Nessuna lista separata da mantenere. Valori attuali:
Cantiere Brusa, Cantiere Brusa (Tenero), Noleggio Brusa, Cantiere Hammer, Cantiere Hammer (Tenero), Cantiere Kohler,
Porto Ragionale Locarno, Porto Locarno Lanca stornazzi, Porto Patriziale Ascona, Porto Minusio Mappo,
Porto Muralto (OVEST), Porto Brissago, Porto Magadino, Porto di Porto Ronco, (altro) scrivere nel stazionamento.

Nel modulo l'ultima voce si mostra come **"Altro"** e rende obbligatorio il campo Ormeggio.

## 4. Flussi

### 4.1 Brusa inserisce una richiesta
Portale → nuova scheda **"Richieste"** → **"Nuova richiesta"** → modulo §2 → Salva.
La richiesta nasce in stato **"Da prendere a carico"**. Finché Epico non l'ha importata, Brusa può modificarla o
eliminarla. Dopo l'import è in sola lettura: le modifiche passano da Epico.

### 4.2 Epico importa (un clic)
AGENDA admin → Portali Clienti → **"Richieste cantieri"** (badge con il numero di richieste nuove).
Per ogni richiesta non importata:

- **"Apri lavoro"**: crea il lavoro con `create_lavoro_mongo_first` copiando i campi §2 (2–11) e aggiungendo:
  - `Clienti: [<ninox_id del cliente del portale>]` (Cantiere Nautico Brusa, configurato sull'utente portale): è il cliente del lavoro, di apertura e di fatturazione; `Cognome` e `Nome` sono solo il proprietario della barca;
  - `Stato del lavoro: "Registrato"`;
  - `Data ora Check-in`: data e ora dell'import (Europe/Zurich), modificabile poi nella scheda lavoro;
  - `Ricevente`: utente admin che importa.
  La richiesta salva il collegamento (`lavoro_uid`, `job_code`) e l'autore dell'import.
- **"Collega a lavoro esistente"**: per i lavori già aperti a mano, si sceglie il codice (`L26xxx`) e la richiesta
  si collega senza creare nulla.
- **"Scollega"** (solo admin, con conferma e registro): per correggere un collegamento sbagliato.

Protezione dai doppi clic: l'import è atomico (`find_one_and_update` con condizione `lavoro_uid: null`).

### 4.3 Aggiornamento automatico della lista Brusa
Numero lavoro, presa a carico, riconsegna e stato **non si copiano** nella richiesta: si leggono dal lavoro collegato
a ogni lettura della lista. Nessun job di sincronizzazione, nessun dato disallineato.

### 4.4 Preventivo da EruFATT
EruFATT, all'emissione di un preventivo o offerta legato al lavoro, manda già il PDF all'AGENDA
(`POST /api/external/erufatt/lavori/{codice}/allegati`, migrazione EruFATT 0055). Da ora la stessa chiamata porta anche i
campi form:

| Campo | Esempio | Note |
|---|---|---|
| `tipo` | `preventivo` \| `offerta` \| `conferma` \| `fattura` \| `acconto` \| `finale` \| `nota_credito` | |
| `numero` | `260012` | |
| `data_documento` | `2026-10-14` | ISO |
| `imponibile` | `1290.00` | Decimale esatto come testo |
| `totale` | `1394.50` | IVA inclusa |
| `valuta` | `CHF` | |

L'AGENDA li salva sul riferimento dell'allegato (`mongo_allegati.erufatt`). Nel portale compare il documento più recente
con `tipo` in (`preventivo`, `offerta`); se ce ne sono più di uno, un indicatore "+n" li elenca tutti.
**Brusa non vede mai fatture, solleciti o altri allegati.**

### 4.5 Excel
Pulsante **"Scarica Excel"** nella scheda Richieste (anche lato admin, per cantiere). File `.xlsx` generato dal backend
(`openpyxl`, già installato), nome `Richieste_Brusa_AAAA-MM-GG.xlsx`:

- colonne nell'ordine del §2 (1–19), intestazioni in grassetto, filtro automatico, riga d'intestazione bloccata;
- date `gg.mm.aaaa`, importi numerici con formato `#'##0.00`;
- righe riconsegnate con fondo **verde** (`C6EFCE`), le altre senza colore;
- rispetta i filtri attivi nella lista (anno, stato, ricerca).

### 4.6 Colori della riga (portale e admin)

| Condizione | Colore | Etichetta |
|---|---|---|
| Non ancora importata | ambra chiaro | Da prendere a carico |
| Importata, senza Check-Out | nessuno | In lavorazione (+ stato AGENDA) |
| `Data / Ora (Check-Out)` valorizzata | **verde** | Riconsegnato |
| Lavoro collegato annullato (`Annullato / Rifiutato`) o cancellato | grigio barrato | Annullato |

## 5. Modello dati (AGENDA, Mongo)

Collezione nuova `cantiere_richieste`:

```json
{
  "_id": "uuid",
  "portal_username": "brusa",
  "numero_richiesta": "R26-0001",
  "data_registrazione": "2026-09-30",
  "fields": {
    "Natante / Veicolo / Macchinario (Tipo) / Oggetto": "Pardo 43",
    "Numero di targa": "TI 1234",
    "Cognome": "Rossi", "Nome": "Mario",
    "Luogo di lavorazione": "Porto Patriziale Ascona",
    "Luogo di Stazionamento": "Pontile B, posto 12",
    "Parte da lavorare": "Tendalino",
    "Tipo di lavorazione": "Sostituire cerniera lato sinistro",
    "Numero di pezzi ritirati": 2,
    "Note": "Chiavi in capitaneria"
  },
  "lavoro_uid": null, "job_code": null,
  "importata_il": null, "importata_da": null,
  "creata_il": "…", "creata_da": "brusa", "modificata_il": "…",
  "eliminata": false,
  "registro": [{"quando": "…", "chi": "…", "azione": "creata|modificata|importata|collegata|scollegata|eliminata", "prima": {}, "dopo": {}}]
}
```

- `numero_richiesta`: progressivo per anno e cantiere (`mongo_counters`), riferimento comodo per Brusa.
- Eliminazione solo logica (`eliminata: true`), mai cancellazione fisica.
- Indici: `(portal_username, eliminata, creata_il)`, `job_code`.

Utente portale (`portal_users`), campi nuovi: `cliente_ninox_id` (cliente di fatturazione, per `Clienti`) e
`richieste_abilitate` (bool). Modificabili in Impostazioni → Utenti portale.

`mongo_allegati`: sul riferimento degli allegati `documento_erufatt` si aggiunge
`erufatt: {tipo, numero, data_documento, imponibile, totale, valuta}` (importi come stringa decimale).

## 6. API (AGENDA)

Portale (JWT portale, `type: portal`; ogni richiesta filtrata per `portal_username` del token):

| Metodo | Percorso | Note |
|---|---|---|
| GET | `/api/portal/richieste?anno=&stato=&q=` | Lista con campi di ritorno e preventivo già risolti |
| POST | `/api/portal/richieste` | Crea; 403 se `richieste_abilitate` è falso |
| PUT | `/api/portal/richieste/{id}` | Solo se non importata (409 altrimenti) |
| DELETE | `/api/portal/richieste/{id}` | Logica; solo se non importata |
| GET | `/api/portal/richieste/form-config` | Lista porti |
| GET | `/api/portal/richieste/export.xlsx?…` | Stessi filtri della lista |
| GET | `/api/portal/richieste/{id}/preventivo/{nome_file}` | Link temporaneo Dropbox; 404 se l'allegato non è preventivo/offerta del lavoro collegato a una richiesta di quell'utente |

Admin (auth admin + permesso lavori):

| Metodo | Percorso |
|---|---|
| GET | `/api/admin/richieste-cantieri?portal_username=&solo_nuove=` |
| POST | `/api/admin/richieste-cantieri/{id}/importa` |
| POST | `/api/admin/richieste-cantieri/{id}/collega` body `{job_code}` |
| POST | `/api/admin/richieste-cantieri/{id}/scollega` |
| GET | `/api/admin/richieste-cantieri/export.xlsx?portal_username=` |

Esterna EruFATT: `POST /api/external/erufatt/lavori/{codice}/allegati` accetta i campi form opzionali del §4.4.

## 7. Sicurezza e regole

- Separazione: un utente portale vede solo le sue richieste e i dati dei lavori collegati a quelle.
- Nessun dato inventato: i campi di ritorno vuoti restano vuoti ("—").
- Registro su ogni richiesta (chi, quando, prima/dopo).
- Importi mai come float: stringhe decimali in Mongo, `Decimal` nei calcoli, formato `CHF 1'290.00` a video.
- Date ISO nel database, `gg.mm.aaaa` a video e in Excel.
- Notifica: a ogni nuova richiesta, push/e-mail agli admin (riuso `push_service`), opzionale.

## 8. Criteri di accettazione

1. Brusa inserisce una richiesta dal telefono in meno di un minuto; i campi obbligatori sono controllati;
   con "Altro" l'ormeggio è obbligatorio.
2. La richiesta compare in AGENDA in "Richieste cantieri"; "Apri lavoro" crea `L26xxx` con tutti i campi compilati,
   cliente Cantiere Nautico Brusa, stato Registrato, Check-in = ora dell'import. Un secondo clic non crea un doppione.
3. Senza ricaricare manualmente (refresh 2 minuti) la riga Brusa mostra numero lavoro e data presa a carico.
4. Emettendo in EruFATT un preventivo legato a quel lavoro, la riga mostra data, numero, importi e il PDF si apre dal
   portale. Una fattura dello stesso lavoro **non** compare.
5. Compilando `Data / Ora (Check-Out)` nel lavoro, la riga diventa verde nel portale e nell'Excel.
6. L'Excel contiene tutte le richieste filtrate, colonne del §2, date e importi formattati, righe verdi.
7. Un secondo utente portale non vede nessuna richiesta di Brusa (test automatico).
8. Test automatici per: creazione/validazione, import atomico, collega/scollega, filtro per utente,
   filtro preventivi (niente fatture), Excel (colonne e colore), campi EruFATT sull'allegato.
