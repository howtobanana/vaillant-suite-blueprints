# Changelog

## 2026-07-04 (Zuständigkeits-Trennung Heizung/Lüftung)
- Heizungs-Steuerung: Fenster-Fallback-Erinnerung entfernt (2 Inputs + Flow).
  Sie duplizierte die Lüftungs-Meldungen, überschrieb per identischem
  Notification-Tag deren Pushes und hatte einen funktionslosen Snooze-Button,
  der wartende Lüftungs-Eskalationen fern-snoozen konnte. Fenster-Pushes
  kommen jetzt ausschließlich aus der Lüftungs-Steuerung.
- Doku: Zuständigkeits-Matrix in beiden Blueprint-Headern und in
  COMPATIBILITY.md (Aktor-Besitz, ein Benachrichtigungs-Absender pro
  Sachverhalt, doppelte Klima-Entity-Konfiguration, Away-Quellen).

## 2026-07-04 (Lüftungs-Steuerung Refactor)
- Fenster-Timer/Eskalation laufen jetzt PRO FENSTER (keine Mehrfach-Meldungen
  mehr, wenn mehrere Fenster offen sind); Klima-, Präsenz-, Kühl- und
  Warm-Checks werden in Langläufern (Eskalation, 24h-Wait) live ausgewertet
  statt mit eingefrorenen Variablen.
- Sommer-Bypass entfernt: Im Sommermodus gibt es keine 10-min-Schließ-Meldung
  mehr (Abend-Durchlüften bleibt still); vergessene Fenster deckt der
  24h-Pfad bzw. der "Außen wärmer"-Alert ab.
- Kühl-Logik rein temperaturbasiert: Kühlungs-Zeitfenster (Start/Ende/
  Zeitplan) und fixe Morgen-Schließ-Erinnerung entfernt — geschlossen wird,
  wenn außen wärmer als innen wird.
- Klima-Räume (5 Toggles) ersetzt durch direkte Auswahl der Abluftfenster
  (`ac_exhaust_windows`).
- Klima-Automatik neu: Bei freier Kühlung Rückfrage-Push mit "Weiterlaufen
  lassen"-Button (Auto-Off nach 10 min ohne Antwort); bei Erreichen der
  Innen-Schwelle (10 min stabil) automatische Abschaltung mit Info-Push.
- Warm-Suppress: Innen-Schwelle nutzt jetzt die Kühl-Innen-Schwelle (ein
  Input weniger); `presence_filter_strict` entfernt (Fallback-Verhalten).
- Away-Erkennung akzeptiert person/group ('not_home'/'off') zusätzlich zu
  zone ('0') — konsistent zum Trigger.
- ⚠️ Breaking: Inputs entfernt/geändert — Automation nach Blueprint-Update
  einmal neu konfigurieren (v. a. Abluftfenster wählen).

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
