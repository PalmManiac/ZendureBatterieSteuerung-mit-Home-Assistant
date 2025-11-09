# CHANGELOG

## v2.0 – Smart Forecast Control (2025-02)
**Großes Funktionsupdate – Fokus auf Preisprognose, intelligente Planung und variable Leistung**

### 🚀 Neue Funktionen
- **Preis-Prognose integriert**
  - Entladung wird jetzt im Voraus geplant, wenn teure Zeitfenster erkannt werden.
  - Wenn der Akku dafür zu wenig Energie hat → automatische „Vorladephase“.

- **Dynamische Mindestenergie**
  - Mindest-Energiebedarf richtet sich nach geplanter Entladeleistung.
  - Kein „leerlaufen“ mehr während teurer Zeitfenster.

- **Variable Lade- und Entladeleistung (0–2400W)**
  - Limits per Slider (`input_number`) einstellbar.
  - Sofortige Übernahme der Werte ins Gerät.

- **Tapering**
  - Leistung sinkt automatisch, wenn SOC nahe Max oder Min kommt → verhindert hartes Abschalten.

- **Neue Debug-Sensoren**
  - Erklären verständlich, warum HA gerade lädt, entlädt oder stoppt.

### ✅ Verbesserungen
- **SOC-Reserven werden sauber eingehalten**
- **Laden bevorzugt PV-Überschuss statt Netzstrom**
- **Netzladen nur noch bei Billigpreis oder für geplante Entladung**
- **Automation wurde komplett neu strukturiert (V4)**
  - Keine toten Pfade mehr
  - Keine Endlos-Zustände
  - Trigger reagiert zuverlässiger

### 🐛 Fehlerbehebungen
- Falsche 600W-Begrenzung entfernt → jetzt 2400W möglich
- „Keine teuren Zeitfenster erkannt“ wurde in manchen Fällen falsch angezeigt → korrigiert
- Fehler bei Vorladen des Akkus behoben
- YAML-Fehler bei `choose/default` beseitigt

### 🔧 Kompatible Änderungen
**Diese neuen Helfer sind erforderlich:**
- `input_number.zendure_schwelle_teuer`
- `input_number.zendure_schwelle_extrem`

**Neue Template-Sensoren:**
- `sensor.zendure_naechstes_teures_zeitfenster`
- aktualisierter Debug-Sensor

**Haupt-Automation ersetzt:**  
`Zendure Akku Automatik (V4 – Prognose, dynamische Mindestenergie, FIX)`

### ✅ Optional empfohlen
- Sofort-Übernahme der Slider:
  - `Zendure – Max Entladeleistung übernehmen`
  - `Zendure – Max Ladeleistung übernehmen`

- Mushroom-Dashboard für kompakte Bedienung

## v2.1 – Notladeschutz & Sommer-Update (Aktuell)
**Neu**
- Notladeschutz: sobald SOC ≤ 8 % → automatische Notladung mit 300 W
- Eigene Slider für Notlade-Leistung und Notfall-SOC
- Sommermodus erweitert:
  - Laden mit PV-Überschuss (mit 50 W Puffer)
  - Laden bei billigen Preisen (<= 0,05 €/kWh)
  - Volllast bei Gratis/negativen Preisen
  - Abends automatische Entladung, sobald PV < 100W
- 5-Min-Watchdog zur Selbstaktivierung

**Verbessert**
- Stabilerer Überschussalgorithmus
- Entladung unabhängig von Prognose
- Keine Tiefentladung mehr möglich
- Grenzen komplett über Dashboard konfigurierbar

**Erfordert**
- 2 neue input_number-Helfer  
  - `zendure_soc_notfall_min`  
  - `zendure_notladeleistung`

---

## v2.0 – Automatik + Prognose
- Dynamische Lade- und Entladeleistung
- Preisbasiert: billig Laden, teuer Entladen
- Ziel-SoC, Reserve-SoC, Leistungsgrenzen über Dashboard
- Prognose-basierte Entladeplanung

## v1.0 – Basic Control (2025-01)
- Laden und Entladen abhängig von PV, Strompreis und SOC
- Fester Grenzwert für Entladeleistung (600 W)
- Keine Prognose-Auswertung
- Kein Vorladen für teure Zeitfenster
- Keine dynamische Leistungsbegrenzung
