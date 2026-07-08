# Sardroid Airtrack — Changelog

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
