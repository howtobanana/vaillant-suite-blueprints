# 🌡️ Vaillant Ultimate Heating Blueprint für Home Assistant
Eine kugelsichere, hochkomplexe und gleichzeitig ressourcenschonende Heizungssteuerung für Vaillant-Brennwertthermen (via myVaillant Integration) und smarte Heizkörperthermostate (TRVs).

Dieses Blueprint ersetzt ein Dutzend einzelner Automatisierungen durch eine Idempotente State-Machine. Es kombiniert Echtzeit-Reaktionen mit einem ausfallsicheren Hintergrund-Heartbeat, schützt die Vaillant-Cloud vor Rate-Limits (API-Sperren) und deinen Raspberry Pi vor Abstürzen.

## 🏗️ Core Architecture: Wie die Automation "denkt"
Das Herzstück dieses Blueprints ist die Hybride Idempotente Architektur. Standard-Automatisierungen scheitern oft an API-Limits oder hängen sich in Endlosschleifen auf. Dieses Blueprint löst das durch drei Ebenen:

Instant Direct-Triggers (Echtzeit): Wenn du ein Fenster öffnest oder einen Temperatur-Slider in HA bewegst, feuert die Automation sofort. Keine Latenz, sofortige Reaktion.

5-Minuten Heartbeat (Self-Healing): Alle 5 Minuten wacht die Automation lautlos im Hintergrund auf. Sie sammelt verlorene Funkpakete ein, heilt asynchrone Zustände nach einem HA-Neustart und gleicht Wetterdaten ab.

Der Idempotente API-Filter: Egal ob durch Fenster, Präsenz oder Heartbeat ausgelöst – die Automation berechnet bei jedem Lauf die perfekte mathematische Zieltemperatur für jeden Raum. Bevor ein Funkbefehl gesendet wird, vergleicht sie: Ist-Zustand == Soll-Zustand?

Ja: Die Automation beendet sich sofort (0 API-Calls, 0 CPU-Last).

Nein: Der Befehl wird gesendet.

#### 🛡️ Hardware & API Safety: Die Automation läuft im mode: queued (max: 5). Selbst wenn ein defekter Sensor hunderte Signale pro Sekunde funkt, werden diese sicher verworfen. Die Vaillant-API wird durch einen intelligenten Debouncer (Verzögerung bei schnellen Slider-Eingaben) vor HTTP 429 (Rate Limits) geschützt. Strikte Jinja2-Type-Safety verhindert Abstürze durch nicht verfügbare Sensoren.

## 🌟 Feature Breakdown
Die Steuerung unterteilt dein Zuhause in intelligente Zonen, die miteinander kommunizieren.

### 👑 1. Master & Follower (Das Zentrum)
Der Master-Raum (z.B. Wohnzimmer): Hier hängt das Vaillant-Wandthermostat. Es diktiert die Vorlauftemperatur und Basis-Logik der Therme.

Follower-Räume (z.B. Flur, Durchgangszimmer): Diese Zonen haben eigene TRVs, folgen aber immer blind dem Master – abzüglich eines einstellbaren Offsets (z.B. Master -2°C). Sie senden niemals eigene Heizanforderungen an die Therme.

### 🚪 2. Isolierte Räume (Räume 2 bis 5)
Schlafzimmer, Büro oder Bad. Diese Räume agieren autark mit eigenen Zeitplänen und Präsenzmeldern.

Dynamic Room Boost: Wenn z.B. das Badezimmer kalt ist, der Master im Wohnzimmer aber eigentlich auf "Eco" gehen würde, funkt das Badezimmer einen "Boost-Bedarf". Der Master hebt seine Temperatur temporär um z.B. +2°C an (einstellbar), bis das Badezimmer warm ist. Danach fällt das System sauber zurück. Kein Flattern!

### 👤 3. Intelligente Präsenz (Restart-Resilient)
Die Anwesenheitserkennung (z.B. via ESPHome/Aqara mmWave Radarsensoren) ist komplett gegen Home Assistant Neustarts gehärtet.

Walk-Through Protection: Ein einstellbares Delay (z.B. 2 Min) stellt sicher, dass die Heizung nicht hochfährt, nur weil du kurz etwas aus dem Raum holst.

Restart-Survival: Nutzt HA's last_changed Attribut. Wenn HA neu startet, während du auf der Couch sitzt, erkennt das Blueprint anhand des Zeitstempels sofort deine reale Anwesenheitsdauer und stellt den Komfort-Modus wieder her.

## 🪟 4. Advanced Window Management & Smart Drop
Virtual & Hardware Sensors: Verknüpft physische Fensterkontakte logisch (ODER) mit virtuellen Fenster-Helfern.

Window Heat Delay: Fenster auf = sofortige Erkennung. Aber erst nach Ablauf des Delays (z.B. 30 Sekunden) dreht die Heizung auf Frostschutz ab. Perfekt für kurzes Stoßlüften ohne Anlagen-Verschleiß.

Smart Drop Erkennung: Fällt die Temperatur im Raum unnatürlich schnell (X °C unter Soll), erkennt die Automation ein "offenes Fenster", selbst wenn kein Sensor verbaut ist, und riegelt das TRV ab.

## 🌤️ 5. Klima- & Feuchtigkeits-Kompensation
Sommer-Modus: Überschreitet die Außentemperatur einen Schwellwert (z.B. 18°C), greift eine Hysterese und die TRVs werden in den Sommerbetrieb gezwungen, um unnötiges Heizen in der Übergangszeit zu vermeiden.

Feuchte-Kompensation: Ist die Luftfeuchtigkeit im Master-Raum zu hoch (> 65%), senkt die Steuerung den Zielwert leicht ab (-0.5°C), um das Raumklima (Taupunkt/Schimmelgefahr) zu optimieren.

## 🆘 6. Global Overrides & Fallbacks
Eco- & Away-Mode: Verlässt du das Haus (Geofencing), greift ein globaler Away-Schalter, der alle Zonen sofort absenkt.

Panic Mode: Fällt die Vaillant-Cloud aus oder Sensoren sterben, übersteuert ein "Panic Helper" das gesamte System und zwingt die Anlage in einen sicheren, warmen Basis-Betrieb.

## 🛠️ Voraussetzungen & Helper
Für den optimalen Betrieb benötigt dieses Blueprint einige (optionale) Helfer (input_boolean, input_number), die du in Home Assistant unter Einstellungen -> Geräte & Dienste -> Helfer anlegen kannst.

Master Last Write Tracker: ZWINGEND ERFORDERLICH! Ein input_datetime Helfer (Datum & Uhrzeit). Hier speichert das Blueprint intern, wann der letzte Funkbefehl an Vaillant ging, um Cooldowns zu berechnen.

Presence Helper (Pro Raum): input_boolean Schalter, in denen die Automation den aktuellen, entprellten Präsenz-Status des Raumes ablegt.

Temperatur-Slider: input_number Helfer, falls du Eco-, Komfort- oder Away-Temperaturen bequem über das HA-Dashboard anstatt hardcodiert einstellen möchtest.

#### (Hinweis: Für die Kommunikation mit der Vaillant Therme wird die myVaillant Integration über HACS vorausgesetzt).

## 💡 Philosophie hinter dem Code ("Bulletproof")
"So simple wie möglich, so komplex wie nötig."

Dieses Blueprint entstand aus der Notwendigkeit heraus, dass herkömmliche Heizungs-Automatisierungen die CPU von Raspberry Pis durch Endlosschleifen überlasten oder Cloud-APIs durch Spamming (Ratelimits) in die Knie zwingen.

Durch das tiefe Type-Safety Auditing der Jinja2-Templates ist das Script immun gegen unavailable oder None States. Die strikte Trennung von Bedarfserfassung (Sensoren/Slider) und Ausführung (API-Write) macht es zu einer echten, industriellen State-Machine für dein Smart Home.