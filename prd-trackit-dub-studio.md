# PRD: Modulo Tracking Lavorazioni ("TrackIt") per Dub Studio Preventivi

## Goal
Aggiungere al gestionale esistente (Dub Studio Preventivi) una nuova sezione "Tracking" che permetta di seguire l'avanzamento di ogni brano di un preventivo approvato, fase per fase (produzione → mix → master), con checklist spuntabili, progress bar automatiche e collegamento allo stato di pagamento già esistente. Non è un clone 1:1 di TrackIt: è una versione semplificata, agganciata ai dati reali dei preventivi.

## Features

- **Accesso**: nuova voce di menu "Tracking" accanto a Preventivo/Gestionale/Tariffe/Impostazioni, protetta dallo stesso PIN già in uso.
- **Avvio tracking**: da un preventivo con `stato` = "approvato" (o superiore), un'azione genera l'oggetto di tracking per quel preventivo. `[NEED: confermare se l'avvio è manuale (bottone "Avvia Tracking") o automatico al passaggio a "approvato"]`.
- **Brani**: titoli estratti automaticamente splittando il campo `brano` (comma-separated) del preventivo; modificabili dalla pagina Tracking; ogni modifica si salva e si sincronizza su GitHub come il resto dei dati (stesso file `dub-data.json`, nuova chiave `trackit`).
- **Fasi per brano, generate dinamicamente dai servizi inclusi nel preventivo** (`selIncl`/`selExtra`/`selSession`):
  - **Fase Produzione**: un elemento checkbox per ogni servizio non-mix/non-master incluso (es. Rec Batterie, Editing Batterie, Rec Chitarre, Bass MIDI, Rec Voci, Editing Voci...).
  - **Fase Mix**: presente solo se il preventivo include un servizio "mix"; checklist mix + contatore revisioni fatte/incluse **per singolo brano** (nuovo campo, distinto dagli attuali `revMixIncluse`/`revMixFatte` di preventivo, che restano invariati per non rompere la logica esistente).
  - **Fase Master**: presente solo se il preventivo include un servizio master_*; stessa logica di Mix (checklist + revisioni per brano).
  - Ogni checklist parte da un default coerente con i servizi acquistati, ma è **liberamente modificabile per singolo brano**: aggiungi/rimuovi elementi senza toccare gli altri brani o il preventivo.
- **Progress bar di fase**: calcolata sugli elementi spuntati di quella fase per quel brano; al 100% la fase è completa e il brano passa alla fase successiva.
- **Card brano**: mostra fase attiva + % completamento fase attiva. Click → apre modale con checklist della fase attiva e tracker revisioni (se fase Mix/Master).
- **Progress bar generale del preventivo**: combina il completamento medio di tutti i brani con lo `stato` di pagamento già esistente (bozza/approvato/acconto/saldato). Al 100% = lavoro consegnato.
- **Deadline (opzionale)**: campo data per brano o per intero preventivo, con pulsante "Esporta .ics" per aggiungerla a un qualsiasi calendario (Google/Apple/Outlook) via file scaricabile — evita l'integrazione OAuth diretta con Google Calendar, molto più complessa su un sito statico senza backend.
- **Persistenza**: stesso meccanismo già in uso — localStorage + sync manuale/automatico su GitHub tramite il token già configurato in Impostazioni, stessa struttura di file.
- **UI**: stessa dark theme del gestionale attuale (variabili CSS esistenti: `--bg`, `--bg2`, `--accent` ecc.), nessun tema separato da mantenere.

## Behaviour
- I dati di tracking vivono in `dub-data.json` sotto una nuova chiave `trackit`, indicizzata per `id` del preventivo — non richiede un secondo file né un secondo push su GitHub.
- Se un preventivo viene modificato dopo l'avvio del tracking (es. cambi i servizi inclusi), le fasi già generate **non** si rigenerano automaticamente per non perdere il lavoro già spuntato — eventuali nuovi servizi vanno aggiunti a mano come elementi extra.
- Il conteggio revisioni per brano è indipendente da quello aggregato del preventivo; se in futuro serve un totale, si somma dai brani.
- Nessun limite di storage particolare: solo testo/checkbox, niente file audio o allegati binari pesanti.

## Out of Scope (per ora)
- Scenari/Workspaces multipli (My Tracks, Label, Mastering Jobs...) — ogni preventivo è già uno scenario naturale, non serve un livello ulteriore.
- Idea Drop con audio player integrato per note vocali/idee.
- Pannello statistiche/analytics avanzato (tempo medio, bottleneck).
- Theme builder personalizzabile per scenario.
- Libreria di template di checklist riutilizzabili cross-progetto (si parte da un default fisso, modificabile per brano, non da una libreria).
- Collegamento diretto alla cartella locale del progetto (Ableton/Logic/FL Studio) — rimandabile a una fase successiva con File System Access API (richiede Chrome/Edge).
- Integrazione OAuth diretta con Google Calendar — sostituita per ora dall'export `.ics`.
