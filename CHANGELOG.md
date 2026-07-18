# Sardroid Airtrack — Changelog

## 1.2.5 - 2026-07-18

### Fix: le impostazioni non persistevano nell'exe compilato (onefile)
- **Bug**: chiudendo e riaprendo l'app compilata (`.exe`) si perdevano flotta (`known_aircraft`), provider dati + credenziali, lingua, poll interval, bbox — ogni modifica salvata dalla UI. In dev da sorgente invece tutto persisteva (dettaglio che smaschera la causa).
- **Causa**: build Nuitka `--onefile` estrae tutto in una cartella temporanea `%TEMP%\onefile_*` e la **cancella alla chiusura**. Il config veniva risolto con `Path(__file__).parent`, che nell'exe onefile punta a quella temp volatile: i salvataggi tecnicamente riuscivano ma sparivano al riavvio. Il config impacchettato va trattato solo come *seed* di default, mai come file scrivibile.
- **Fix** in [airtrack_ui.py](airtrack_ui.py): nuova `_resolve_config_path()`. Se si gira come exe compilato (`sys.frozen` / `__compiled__`), il config scrivibile vive in `%APPDATA%\SardroidAirtrack\airtrack.config.json`, seedato al primo avvio dal default impacchettato. In dev da sorgente il comportamento storico resta invariato (config nel progetto, `.local` preferito se presente). Fallback sul seed se `%APPDATA%` è assente/non scrivibile. Correzione in un unico punto: tutti i salvataggi UI usano già `self.config_path` derivato da lì.
- **Nota utenti già installati**: al primo avvio della 1.2.5 ripartono da config seed in `%APPDATA%`. Le loro modifiche precedenti erano comunque già perse a ogni chiusura, quindi non c'è nulla da migrare.

### Fix: modale "Area" tagliava i parametri in basso
- Il dialog bbox aveva la mappa (`expand=True`) packata prima di filtri e toolbar: con finestra troppo bassa i parametri Discovery e i bottoni venivano spinti fuori dal bordo inferiore e tagliati. Ora filtri + toolbar sono packati dal basso (`side="bottom"`) prima della mappa, così è la mappa a comprimersi, non i parametri. Aggiunto `minsize(820, 680)`.

### Poll interval: 15/30/60 (era 20/30/60)
- Il valore minimo selezionabile scende da 20s a **15s** (allineato al `min_poll_seconds` dei feed comunitari readsb-based). Predefinito invariato a 30s.

## 1.2.4 - 2026-07-11

### Fix: tasto "Avvia" inefficace dopo "Ferma" se il worker precedente e' ancora appeso
- **Bug**: dopo aver premuto **Ferma**, se per qualsiasi motivo il thread `PollerWorker` era ancora bloccato (es. `fetch_states` in retry lungo su rete lenta, POST verso Sardroid in timeout), premere **Avvia** non riavviava il poll. Motivo: `_on_start` faceva `if self.poller and self.poller.is_alive(): return` — bloccante anche se `stop()` era gia' stato invocato.
- **Fix** in [airtrack_ui.py::_on_start()](airtrack_ui.py): il check ora ignora un worker che ha gia' ricevuto `stop()` (`_stop_flag.is_set()`). Se il worker precedente e' ancora vivo *ma* fermato, ne creiamo comunque uno nuovo. Il vecchio e' un thread daemon: uscira' da solo quando la chiamata bloccante ritorna, o al piu' tardi alla chiusura app.

## 1.2.3 - 2026-07-11

### Fix: versione app non aggiornata dopo auto-update
- Stesso bug osservato in Sardroid Server v9.0.7. `get_current_version()` leggeva prima `Path(sys.executable).parent/VERSION` (esterno all'exe, non toccato dall'auto-update) e solo dopo `Path(__file__).parent/VERSION` (bundle Nuitka).
- Fix: inverto l'ordine — prima il bundle interno, poi il fallback esterno. Cosi' dopo un auto-update la versione nell'header risulta allineata all'exe reale.

## 1.2.2 - 2026-07-11

Rilascio di manutenzione + omogeneizzazione release pipeline.

### Fix: aggiornamento.json non piu' verboso
- `release.ps1` ora cap-pa le note di release a **6 bullet massimo** invece di estrarre tutti quelli del CHANGELOG.
- Aggiunta automatica di `changelog_url` che punta al CHANGELOG completo su GitHub, cosi' chi vuole leggere tutto lo storico esteso ha il link nella modal update ("Note complete →").
- Aggiornato `build_exe.bat` (opzionale, non ancora applicato: `--nofollow-import-to=PIL.ImageQt` per silenziare warning informativo Nuitka).

## 1.2.1 - 2026-07-11

Fix UX minori + differenziazione visiva rispetto ad Aistrack.

### Fix: rate_label non piu' vuota per provider senza contatore numerico
- La label in alto a destra ("OpenSky · N req rimaste") diventava **completamente vuota** quando l'utente selezionava un provider senza `remaining` esposto negli header (adsb.fi, airplanes.live, ricevitore ADS-B locale). Il vecchio `_RateLimitProxy` iterava tutte le istanze e il primo che aveva `remaining=None` faceva sparire la label.
- Fix: la label ora legge direttamente dal provider corrente tramite `core.get_provider(self.cfg).rate_limit_snapshot()`. Se il provider non ha un contatore numerico, mostra `display_name · quota_label` (es. "Ricevitore ADS-B locale (readsb/dump1090) · no rate limit (LAN)").
- Copre tutti i 6 provider: comportamento vecchio per OpenSky/FlightAware (contatore numerico verde/giallo/rosso), fallback descrittivo per adsb.fi/airplanes.live/readsb_local/aviationedge.

### Differenziazione visiva rispetto ad Aistrack
- Aggiunta **striscia identificativa rossa VVF alta 6px in cima alla finestra**. Marker sempre visibile per distinguere Airtrack a colpo d'occhio da Aistrack (che ha una striscia ciano).
- **AppUserModelID esplicito** su Windows (`PompiereCattivo.SardroidAirtrack.1.2`): senza questo, Windows raggruppa le finestre Python di Airtrack e Aistrack sotto un'unica voce nella barra applicazioni. Con l'AUMID esplicito ognuno ha il proprio spazio in taskbar + start menu + jumplist.

## 1.2.0 - 2026-07-11

Filtri famiglia Noti/Sconosciuti, astrazione DataProvider generica, dialog "Fornitore dati" riscritto per supportare piu' fornitori.

### Filtri famiglia (Noti / Sconosciuti)
- Due nuove checkbox nella toolbar: **Noti** e **Sconosciuti** (default entrambi on).
- Agiscono sia sulla UI (righe nascoste nella tabella aeromobili) sia sull'ingest (le entry filtrate NON vengono POST-ate a Sardroid). Un aeromobile in anagrafica passa se "Noti" e' on, un aeromobile fuori anagrafica passa se "Sconosciuti" e' on.
- Auto-save on-change nelle chiavi `discovery.include_known` / `discovery.include_unknown` del config.
- Guardrail UX: disattivare entrambi mette il poller in muto, quindi Airtrack riaccende automaticamente "Noti" con log di warning.
- Toolbar riorganizzata su due righe per compensare l'aggiunta dei toggle. Poll interval e i due filtri stanno nella seconda riga.

### DataProvider generico + refactor
- Nuovo package [`providers/`](providers/) con l'astrazione `DataProvider` (`fetch_states`, `rate_limit_snapshot`, `test_connection`, `credentials_schema`, `min_poll_seconds`).
- OpenSky spostato in `providers/opensky.py` come primo provider concreto (`OpenSkyProvider`). Nessuna modifica funzionale: OAuth2, Basic, anonimo, retry token, tracking rate limit sono tutti li'.
- `airtrack.py::fetch_states(cfg)` sceglie il provider corrente in base a `cfg["source"]["type"]` (default `"opensky"`). Alias `fetch_opensky` mantenuto per retro-compat.
- `state_to_entry` refactor: ora accetta un `AircraftState` dict normalizzato (chiavi `icao24`, `lat`, `lon`, `altitude_m`, `speed_ms`, ...). Retro-compat con liste OpenSky per test/POC standalone.
- Il campo `extra.source` propagato al server e' preso da `cfg["source"]["type"]` (era hardcoded `"opensky"`), cosi' Sardroid puo' distinguere l'upstream.
- `last_rate_limit` diventa un proxy che legge dal provider corrente al momento dell'accesso (retro-compat con `getattr(core, "last_rate_limit")`).

### Provider concreti aggiuntivi (6 totali in 1.2.0)
- **adsb.fi** (`providers/adsbfi.py`) — feed comunitario ADS-B, nessuna auth. Base `https://opendata.adsb.fi/api/v2`.
- **airplanes.live** (`providers/airplaneslive.py`) — altro feed comunitario ADS-B, nessuna auth. Base `https://api.airplanes.live/v2`.
- **FlightAware AeroAPI** (`providers/flightaware.py`) — servizio commerciale con free trial. Auth via header `x-apikey`. Endpoint bbox nativo `/flights/search/positions`. Altitudine in centinaia di piedi (FL), timestamp ISO 8601. `min_poll_seconds = 30`.
- **Aviation Edge** (`providers/aviationedge.py`) — commerciale con free trial ~100 req/giorno. Auth via query-string `?key=`. Nessun endpoint bbox nativo, filtro client sul risultato globale. Struttura JSON annidata (`geography.altitude`, `speed.horizontal`), altitudine gia' in metri, velocita' in km/h. `min_poll_seconds = 60`.
- Nuovo `providers/_readsb_common.py` con logica condivisa per i due feed comunitari (readsb-based): `bbox_to_center_radius()`, `normalize_readsb_state()`, `extract_now_epoch()`. Un solo posto per conversioni ft→m, knots→m/s, ft/min→m/s.
- I due provider readsb-based convertono il `discovery.bbox` in cerchio circoscritto (capped 250 nm) al centro geometrico, perche' le loro API v2 accettano `lat/{lat}/lon/{lon}/{radius_nm}`. Eventuali aeromobili raccolti fuori dal bbox reale vengono filtrati a valle.
- Rate limit soft ~1 req/s per adsb.fi/airplanes.live → `min_poll_seconds = 15`. Alla selezione via dialog, il poll interval viene bumpato automaticamente al valore piu' basso >= min_poll_seconds tra i preset (15/30/60).
- Il dialog "Fornitore dati" gestisce automaticamente tutti e 5 senza modifiche: e' completamente pilotato da `credentials_schema` (nessun codice hardcoded per provider).
- `test_connection` implementato per tutti e 5: chiamata leggera per verificare auth+connettivita' inline nel dialog.
- Test A/B live tra adsb.fi e airplanes.live (bbox Piemonte): lo stesso volo (SWR29K, icao24 `4b1803`) rilevato da entrambi con quota entro <10m di differenza e speed identica — l'astrazione `DataProvider` regge, gli `AircraftState` sono interscambiabili.
- **FlightAware e Aviation Edge sono NON testati live**: entrambi non offrono un trial gratuito accessibile senza carta di credito e piano attivo, quindi durante lo sviluppo non e' stato possibile verificare che i dati reali corrispondano alla documentazione dell'API. Il codice segue le specifiche pubbliche ma **potrebbe richiedere rifiniture** al primo uso reale (es. campi rinominati, formato timestamp diverso, ordine di risposta). Chi ha una API key valida puo' testare con "Test connessione" nel dialog e segnalare eventuali discrepanze.
- **Ricevitore ADS-B locale** (`providers/readsb_local.py`) — nuovo provider per feed self-hosted. Si connette a un endpoint HTTP `aircraft.json` esposto da readsb / dump1090 / PiAware / tar1090 sulla LAN. Nessuna auth. Riusa il modulo `_readsb_common.py`. Config: `source.endpoint_url` (default `http://localhost:8080/data/aircraft.json`). Vantaggi: latenza <1s, no rate limit, no dipendenza da internet, copertura precisa della zona d'interesse. Ideale se si installa un Raspberry Pi con dongle SDR (~50€ una tantum).

### Dialog "Fornitore dati" (rinominato da "OpenSky")
- Il bottone toolbar "🔑 OpenSky" diventa "📡 Fornitore dati".
- Il vecchio `airtrack_account_dialog.py` (specifico OpenSky) e' sostituito da `airtrack_provider_dialog.py` generico.
- Dropdown "Provider" in cima: oggi mostra solo OpenSky ma pronto ad accogliere nuovi provider (adsb.fi, airplanes.live, ...) semplicemente aggiungendoli al registro `providers/PROVIDERS`.
- Dropdown "Modalita' auth" con 3 gruppi per OpenSky (Nessuna / OAuth2 / Basic). I field vengono generati dinamicamente dal `credentials_schema` del provider.
- Tasto "Test connessione" usa `provider.test_connection()` in thread separato con risultato inline.
- Import di `credentials.json` mantenuto per OpenSky OAuth (bottone visibile solo quando la modalita' selezionata usa i campi `oauth.client_id`/`oauth.client_secret`).
- Password toggle 👁 per tutti i campi marcati `secret: True`.

### Retro-compatibilita'
- Config esistenti con `source.type = "opensky"` + credenziali OAuth/Basic funzionano identici a prima.
- Caller esterni che usano `core.fetch_opensky` o `core.state_to_entry(list, ...)` continuano a funzionare.
- La release 1.2.0 non introduce provider nuovi (solo scaffold): l'utente vede lo stesso flusso OpenSky, con UI leggermente diversa.

## 1.1.0 - 2026-07-08

Filtri Discovery editabili + soglie alert + installer pulito.

### Filtri Discovery
- Il dialog "🗺️ Area" ora contiene anche i filtri Discovery editabili direttamente dalla UI (prima solo da file config): quota massima (m), velocità minima (kn), escludi aeromobili a terra.
- Aggiunte due soglie di alert configurabili:
  - **Alert area bbox** (km², default 100 000): al salvataggio del bbox, se l'area supera la soglia viene mostrato un dialog di conferma. Guardrail contro configurazioni che sondino aree troppo ampie e stressino OpenSky/Sardroid.
  - **Alert aeromobili per poll** (default 500): se un poll ritorna più aeromobili della soglia, la riga stats in barra di stato diventa arancione con prefisso `⚠ TROPPI AEROMOBILI (>N)`. Advisory, non blocca il poll.
- Le soglie non bloccano mai il flusso: sono avvisi per aiutare a evitare configurazioni troppo aggressive.
- Chiavi i18n IT/EN/ES per tutti i nuovi testi.

### Installer pulito
- `airtrack.config.json` distribuito nell'installer parte con `known_aircraft: []` — nessuna flotta pre-configurata. Ogni nuova installazione è pulita.
- La cartella `presets/` distribuita nell'installer contiene solo un README esplicativo — nessun preset di flotta reale.
- Le flotte reali di sviluppo (es. flotta VVF italiana) sono spostate in `_dev_presets/` locale, escluso da installer, build, backup e git.
- `SECRET GUARD` in `release.ps1` esteso: la release viene bloccata se il config sorgente contiene `known_aircraft` non vuoto o se `presets/` contiene file `.json`, prevenendo leak di dati sensibili come già accade per le credenziali.

## 1.0.9 - 2026-07-06

Security fix.

- Rimosse credenziali OpenSky (client_id, client_secret) dal file `airtrack.config.json` distribuito.
- Introdotto override runtime `airtrack.config.local.json` per credenziali di sviluppo (escluso da build, backup, git).
- Aggiunto guard nel `release.bat` che blocca la pubblicazione se il config sorgente contiene credenziali reali.
- Le release 1.0.0 / 1.0.1 / 1.0.3 sono state rimosse: contenevano credenziali OpenSky dell'autore, ora ruotate.

## 1.0.3 (rimossa - security)

Prima versione. Migrazione del plugin da `sardroid_server/tools/airtrack/` a progetto indipendente in `D:\sardroid_airtrack\`.

- UI Tkinter con toolbar 🔑 OpenSky, 🗺️ Area, 🛩️ Flotta, 📥 Importa, 📤 Esporta preset, 🌐 Lingua
- Auth: Basic auth OpenSky + OAuth2 client credentials (con import da `credentials.json`)
- Discovery bbox visuale con toggle modalità disegno
- Anagrafica flotta editable con auto-lookup adsbdb.com
- Sweep esplicito: al poll successivo, i tracker fuori bbox spariscono da Sardroid
- Rate limit remaining visibile nella UI
- Doppio livello: sweep dal plugin (immediato) + safety net server (60s)
- **i18n**: interfaccia in IT / EN / ES, campo `language` in config, selettore in toolbar (richiede riavvio per applicare)
- **Multi-target Sardroid**: `sardroid_url` può essere una stringa (single-target) o una lista. Se sono configurati dev + prod, Airtrack sonda entrambi e inietta su quelli up. Re-probe automatico ogni 30s: se un Sardroid parte dopo Airtrack (o si spegne), il fanout si adatta al prossimo poll.
- **Icona applicazione**: TAZ (stessa di Sardroid Server) usata come icona exe, finestra, shortcut Start Menu, installer.
- **Build concatenata**: `build_exe.bat` produce ora anche l'installer se Inno Setup 6 è disponibile (single-shot: exe + setup).
- Batch di supporto: start_dev, backup, run
