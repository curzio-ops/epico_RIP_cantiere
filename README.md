# epico_RIP_cantiere — Richieste di riparazione dai cantieri

Modulo del **Portale BRUSA** dell'AGENDA: il Cantiere Nautico Brusa registra ogni riparazione di selleria
da fare; Epico la importa con un clic come lavoro AGENDA; la lista del cantiere si aggiorna da sola
(numero lavoro, presa a carico, riconsegna, preventivo EruFATT) e si scarica in Excel.

Il codice vive nell'AGENDA (sviluppata con Emergent) e, per la parte preventivi, in EruFATT.
Questo repo contiene la specifica e il prompt da dare a Emergent, versionati.

| File | Contenuto |
|---|---|
| [`docs/SPECIFICA.md`](docs/SPECIFICA.md) | Specifica funzionale e tecnica, campi, flussi, criteri di accettazione |
| [`docs/PROMPT_EMERGENT.md`](docs/PROMPT_EMERGENT.md) | Prompt pronto da incollare in Emergent (AGENDA) |
| [`docs/DECISIONI.md`](docs/DECISIONI.md) | Registro delle decisioni |

## Stato

- EruFATT: invio dei dati del preventivo (tipo, numero, data, importi) insieme al PDF — pronto sul branch
  `claude/brusa-preventivi-agenda` di `epico_FATTURA`.
- AGENDA: da implementare con Emergent (prompt in `docs/PROMPT_EMERGENT.md`).
