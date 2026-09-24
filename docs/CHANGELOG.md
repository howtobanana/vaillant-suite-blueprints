# Changelog

## 2026-09-24 (Heizung: Review — Sommer-Erkennung, Bugfixes, Aufräumen)
- **Sommer-Erkennung vereinfacht (Breaking für die Einstellungen):** Statt
  10 nur noch 3 Regler — Schalter, Außen-Mittelwert-Sensor (Statistics-
  Helfer, 72 h) und Heizgrenze (Default 15 °C). Sommer EIN bei ≥ Heizgrenze
  +1,5 °C, AUS bei < Heizgrenze −1,5 °C. Entfallen: Automatik-Schalter
  (Automatik = Sensor gewählt), Innen-Referenz, Innen-Minimum, EIN-/AUS-
  Schwelle, Haltezeit, Start-/Endmonat. Kein Fallback mehr auf den
  Roh-Außensensor. Der Abgleich läuft in jedem Lauf statt über
  Flanken-Trigger — übersteht HA-Neustarts (vorher verschluckte ein
  Neustart während der Haltezeit die Umschaltung, und mit Roh-Sensor riss
  die 24-h-Haltezeit täglich ab → Sommermodus blieb hängen).
- **Sommer schaltet die Heizung immer ab:** "Sommer: Heizung aus (WW bleibt)"
  und "Sommer-Aus über Preset" entfernt (HVAC 'Off' der myVaillant-Zone
  lässt das Warmwasser laufen). Die Sommer-Sektion hat damit 3 Regler.
- **Keine Alert-Pushes mehr aus der Heizung:** Kritische Zustände erscheinen
  nur noch als HA-Meldung (verschwindet von selbst). Offline-Pushes kommen
  aus der System-Diagnose (siehe unten, ein Absender pro Sachverhalt);
  NOT-AUS und API-Budget regeln sich selbst. Beseitigt den Push-Spam bei
  jedem Fenster-/Präsenz-Event, solange ein Problem bestand.
- **System-Diagnose: TRV/Sensor-Offline-Push repariert.** Der Zweig war
  unerreichbar (verlangte `main_loop` und zugleich nicht `main_loop`) und
  verließ sich auf die Heizung. Jetzt genau ein Push pro Ausfall: ein Gerät
  zählt nur im 30-min-Fenster nach der Wartezeit als offline.
- `source_url` aller Blueprints zeigte auf den nicht existierenden Branch
  `main` (404) → jetzt `vaillant-suite`. In HA liegen Diagnose und Lüftung
  jetzt unter den Repo-Dateinamen (`vaillant_system-diagnose.yaml`,
  `vaillant_lüftungs-steuerung.yaml` statt `vaillant_health_monitor.yaml` /
  `vaillant_ventilation.yaml`); Einstellungen unverändert.
- Startschutz (120 s nach HA-Start/Reload) gilt jetzt für jeden Trigger;
  vorher konnte ein time_pattern- oder Fenster-Lauf ihn per mode: restart
  umgehen.
- Im Sommer-Aus kein "Schreiben erlaubt" und kein Leitstand-Event alle
  3 min mehr (Master 'off' → Soll 0 wurde als Anhebe-Bedarf gewertet).
- "TRV Write Failed" verschwindet wieder, sobald das TRV antwortet;
  Readback wartet bis 15 s statt fix 3 s.
- **Therme-Klima-Verriegelung entfernt** (3 Inputs, 2 Trigger, Lockout-Flow).
  Nur für raumluftabhängige Thermen relevant, und eine softwareseitige
  CO-Schutzfunktion ist bei Fehlkonfiguration ein Haftungsrisiko. Der
  letzte Stand mit Feature ist als Tag
  `snapshot-pre-remove-therme-lockout-2026-09-24` gesichert
  (`git show <tag>:blueprints/vaillant_heizungs-steuerung.yaml`).
- Aufgeräumt ohne Verhaltensänderung: Präsenz-Branches (10 → 1),
  Schreibpfade A/B/C (3 → 1, `_write_path` als einzige Entscheidung — das
  Debug-Log zeigt jetzt den echten Pfad), Leitstand aus gemeinsamen
  Bausteinen, Notify-Liste und Master-Ist-Temperatur nur noch einmal,
  Leer-Prüfung zentral (`_empty`), toter Code entfernt. Insgesamt ~4040 → ~3400 Zeilen.

## 2026-07-05 (Lüftung: Totband gegen widersprüchliche Meldungen)
- "Zeit zum Lüften" und "Fenster zu — zu warm draußen" nutzen jetzt dieselbe
  Temperatur-Linie (innen + Kühl-Offset); der "zu warm"-Alert feuert erst
  1 °C darüber (Totband). Vorher konnten sich beide bei fast gleichen
  Temperaturen direkt widersprechen (empfehlen → öffnen → "zu warm").
- 4b-Meldungen zeigen konfigurierte Raum-Namen statt der friendly_names der
  Fensterkontakte ("Fenster, Fenster").
- 4b Stage 1 ohne time-sensitive — keine Fokus-/Schlafmodus-Durchbrechung.

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
