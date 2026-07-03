# Changelog

## 2026-07-04
- Heizungs-Steuerung: Sommermodus schaltet optional die Therme-Heizung ab
  (HVAC 'Off', Warmwasser läuft weiter) und setzt alle TRVs auf das
  TRV-Minimum statt weiterzuregeln. Neue Inputs `summer_heat_off_enabled`
  (Default an) und `summer_off_preset` (opt).
- Heizungs-Steuerung: Soll-0-Guard — Master-Writes werden übersprungen, wenn
  die Vaillant-Entity kurzzeitig Soll 0 meldet (Cloud-Poll-Artefakt), statt
  gegenzuregeln.
- Heizungs-Steuerung: Review-Fixes zum Sommer-Aus — optionaler
  `summer_off_preset` wird beim Saisonende zurückgesetzt (`none`); PATH S
  prüft den Master-Zustand live statt per Lauf-Start-Variable (kein
  3-min-Blip nach der Klima-Verriegelung) und verifiziert die Abschaltung
  mit Warnung im Log; Leitstand zeigt im Sommer "Heizung aus (WW läuft)"
  bzw. Status SOMMER statt irreführender Sollwerte.
- Heizungs-Steuerung: Therme-Klima-Verriegelung fachlich korrigiert:
  softwareseitig ist kein komplettes Ausschalten möglich (Presets beenden nur
  Heiz-/WW-Programme); Feature nur für raumluftabhängige Geräte relevant,
  raumluftunabhängige Brennwertgeräte dürfen parallel zur Klima laufen.
  `therme_off_preset` Default bleibt leer (climate.turn_off).

## 2026-02-13
- Repository initial versioned (`baseline-2026-02-13`).
- Suite neu strukturiert in `blueprints/`, `releases/legacy/`, `docs/`.
- Dateinamen auf deutsch harmonisiert:
  - `vaillant_heizungs-steuerung.yaml`
  - `vaillant_lüftungs-steuerung.yaml`
  - `vaillant_geofencing.yaml`
  - `vaillant_system-diagnose.yaml`
- `source_url`-Header auf neue Pfade angepasst.
