# Kompatibilität der Suite

## Architektur
- `vaillant_heizungs-steuerung.yaml` ist das zentrale Backbone.
- `vaillant_geofencing.yaml` steuert den Away-Status (Helper-basiert).
- `vaillant_lüftungs-steuerung.yaml` ist unabhängig von der Kern-Heizlogik.
- `vaillant_system-diagnose.yaml` überwacht und benachrichtigt, greift nicht direkt in die Kernsteuerung ein.

## Zuständigkeits-Matrix

| Domäne | Heizungs-Steuerung | Lüftungs-Steuerung |
|---|---|---|
| Wärme-Aktorik (Therme + TRVs) | steuert | — |
| Fenster → Frostschutz | steuert (still, ohne Push) | — |
| Fenster → Benachrichtigungen | — | einziger Absender |
| Saison (Sommermodus-Boolean) | Single Writer | liest nur |
| Kühl-Empfehlungen | — | sendet |
| Mobile Klima als Sensor (läuft sie?) | — | Abluftfenster-Ausnahme |
| Mobile Klima als Aktor (ausschalten) | — | Klima-Automatik (Auto-Off) |
| System-Health (offline, Budget, NOT-AUS) | sendet | — |

Prinzip: Wer den Aktor besitzt, steuert ihn; pro Sachverhalt gibt es genau
einen Benachrichtigungs-Absender (keine Doppel-Pushes).

## Integrationsprinzip
- Gemeinsame Helper/Entities zwischen Modulen müssen konsistent konfiguriert werden.
- Geofencing und Core sollen denselben Away-Helper verwenden.
- Die Klima-Entity wird in Heizung (Verriegelung) und Lüftung (Abluft/Auto-Off)
  separat konfiguriert — beide müssen auf dieselbe Entity zeigen.
- Away-Quellen: Heizung nutzt den Away-Helper (via Geofencing/Urlaubsschalter),
  Lüftung liest die Anwesenheits-Zone direkt. Konsistent, solange Geofencing
  den Helper aus derselben Zone speist (kleine Latenz möglich).

## Versionsmanagement
- Modulversionen über Git-Tags dokumentieren.
- Keine Versionssuffixe in Dateinamen verwenden.
