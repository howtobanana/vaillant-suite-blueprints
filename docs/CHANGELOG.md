# Changelog

## 2026-07-01
- Heizungs-Steuerung: Sommermodus kann die Therme-Heizung abschalten (HVAC
  'Off'), Warmwasser läuft weiter, Frostschutz bleibt. Neue Inputs
  `summer_heat_off_enabled` (Default an) und `summer_off_preset` (opt).
- Heizungs-Steuerung: CO-Verriegelung schaltet die Therme jetzt per Default
  komplett ab — `therme_off_preset` Default auf `system_off` (Heizung + WW),
  Hilfetext geschärft.

## 2026-02-13
- Repository initial versioned (`baseline-2026-02-13`).
- Suite neu strukturiert in `blueprints/`, `releases/legacy/`, `docs/`.
- Dateinamen auf deutsch harmonisiert:
  - `vaillant_heizungs-steuerung.yaml`
  - `vaillant_lüftungs-steuerung.yaml`
  - `vaillant_geofencing.yaml`
  - `vaillant_system-diagnose.yaml`
- `source_url`-Header auf neue Pfade angepasst.
