# Sardroid Airtrack — Changelog

## 1.0.5 - 2026-07-06

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
