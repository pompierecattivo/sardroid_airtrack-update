# Sardroid Airtrack — Update Channel

Canale di distribuzione dell'installer di **Sardroid Airtrack**, il plugin desktop
Windows che alimenta [Sardroid Server](https://github.com/pompierecattivo/sardroid_server-update)
con posizioni di aeromobili prese da [OpenSky Network](https://opensky-network.org).

Questo repository contiene **solo** le release binarie e il manifest di update.
Il codice sorgente non è pubblicato.

## Download

L'installer più recente è sempre disponibile a:

<https://github.com/pompierecattivo/sardroid_airtrack-update/releases/latest/download/SardroidAirtrack_Setup.exe>

Requisiti:

- Windows 10 o 11 (x64)
- Sardroid Server in esecuzione sulla stessa macchina (`https://localhost:8090`, o multi-target)
- Account gratuito su [OpenSky Network](https://opensky-network.org/index.php?option=com_registration)

Installazione: doppio-click sull'`.exe`, il wizard chiede se installare per
l'utente corrente (nessun admin) o per tutti gli utenti (richiede admin).

## Manifest di update

Airtrack, all'avvio, controlla `aggiornamento.json` in questo repository per
mostrare un banner "nuova versione disponibile" quando la versione remota è
superiore a quella locale.

Schema:

```json
{
  "versione": "1.0.0",
  "url": "https://github.com/pompierecattivo/sardroid_airtrack-update/releases/latest/download/SardroidAirtrack_Setup.exe",
  "note": [
    "riga 1 delle note",
    "riga 2 delle note"
  ]
}
```

## Release e changelog

Vedi la [pagina Releases](https://github.com/pompierecattivo/sardroid_airtrack-update/releases)
per la lista completa delle versioni e le note di rilascio. Un [CHANGELOG.md](CHANGELOG.md)
sintetico è mantenuto anche in questo repository.

## Progetti correlati

- [Sardroid Server](https://github.com/pompierecattivo/sardroid_server-update) — il server di destinazione
- Aistrack — plugin gemello per traffico navale (in progetto, non ancora rilasciato)
