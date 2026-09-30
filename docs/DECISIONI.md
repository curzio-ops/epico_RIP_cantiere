# Registro delle decisioni

| Data | Decisione | Motivo |
|---|---|---|
| 30.09.2026 | Modulo richieste dentro l'AGENDA (Portale BRUSA), implementato da Emergent; questo repo tiene specifica e prompt | Stesso database e stessi campi dei lavori: import senza mapping, stato sempre aggiornato senza sincronizzazioni. Il codice AGENDA non si tocca fuori da Emergent |
| 30.09.2026 | I campi della richiesta usano i nomi esatti dei `fields` dei lavori AGENDA | Richiesta CBOR: nessun mapping all'import |
| 30.09.2026 | Numero lavoro, presa a carico, riconsegna e stato letti dal lavoro collegato al momento della lettura, non copiati nella richiesta | Un solo dato vero; niente righe disallineate |
| 30.09.2026 | Riga verde quando il lavoro ha `Data / Ora (Check-Out)` | Scelta CBOR: criterio oggettivo |
| 30.09.2026 | Preventivo: EruFATT manda all'AGENDA, insieme al PDF già allegato al dossier, tipo, numero, data, imponibile, totale e valuta del documento | I preventivi esistono già in EruFATT e il PDF arriva già nel dossier (migr. 0055); servono i dati leggibili per le colonne |
| 30.09.2026 | Il cantiere vede solo preventivi/offerte, mai fatture o altri allegati | Riservatezza: il portale mostra solo ciò che serve al cantiere |
