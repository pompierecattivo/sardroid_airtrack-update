# Sardroid Airtrack — Changelog

## 1.0.0 — 2026-07-06

Prima release pubblica.

### Core
- Poller OpenSky con auth **OAuth2 client credentials** (raccomandato) e **Basic auth**.
- Filtri configurabili: bbox, quota massima, escludi a terra, velocità minima.
- Sweep esplicito verso Sardroid Server: al poll successivo, i velivoli fuori bbox spariscono dalla mappa senza aspettare il timer di 60 s server-side.
- **Multi-target Sardroid**: `sardroid_url` accetta stringa o lista. Fanout automatico su tutti i target attivi; re-probe ogni 30 s per gestire target che salgono/scendono a runtime (es. dev + prod).

### Interfaccia
- UI Tkinter con toolbar OpenSky / Area / Flotta / Preset / Lingua.
- **i18n**: italiano, inglese, spagnolo. Cambio lingua **a caldo** senza riavvio (Poller resta attivo).
- Editor visuale del bbox su mappa Leaflet.
- Editor visuale della flotta con auto-lookup ICAO24 da [adsbdb.com](https://www.adsbdb.com).
- Import / export preset flotta in JSON.
- Rate limit OpenSky visibile in barra di stato.
- Icona applicazione coerente con Sardroid Server.

### Distribuzione
- Installer Inno Setup **dual-mode**: single-user (nessun admin) o all-users (admin), a scelta.
- Notifica update in-app: banner con link diretto al download quando disponibile una versione più recente.
- Backup progetto integrato (`backup.bat`).
- Build concatenata: `build_exe.bat` produce exe Nuitka + installer Inno Setup in un unico lancio.
