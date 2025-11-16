# Zendure Home Assistant – Automatische Akku-Steuerung

Dieses Projekt enthält eine vollständige, intelligente Steuerung für Zendure-Akkus über Home Assistant.  
Die Steuerung berücksichtigt:

✅ PV-Erzeugung  
✅ Haushaltsverbrauch  
✅ Tibber-Strompreis in Echtzeit  
✅ 15-Minuten-Preisprognose  
✅ SOC-Zusteuerung (Reserve / Max)  
✅ variable Lade- und Entladeleistung (0–2400 W)

Das Ziel: **Maximale Ersparnis**, minimale Netzeinspeisung, automatische Akku-Entladung bei hohen Strompreisen und Laden bei Überschuss oder Billig-Zeiten.

---

## 📄 Anleitung
Die vollständige Setup-Dokumentation befindet sich in:

➡ **`zendure_homeassistant_komplett.md`**

Sie enthält:
- Installation
- Template-Sensoren
- Helfer (`input_number`)
- Haupt-Automation (V6)
- Sofort-Übernahme der Slider
- Debug-Auswertungen
- Dashboard-Konfiguration

---

## 🆕 Änderungen & Versionen
Alle Funktionsänderungen und Verbesserungen werden hier dokumentiert:

➡ **`CHANGELOG.md`**

---

## 🧩 Voraussetzungen
Damit die Automatik funktioniert, müssen folgende Entitäten vorhanden sein:

- Tibber-Integration mit Preis & Prognose
- PV-Leistungs-Sensor (z. B. Wechselrichter)
- Netzbezug / Einspeisung
- Zendure-Entitäten:
  - `select.solarflow_2400_ac_ac_mode`
  - `number.solarflow_2400_ac_input_limit`
  - `number.solarflow_2400_ac_output_limit`
  - `sensor.solarflow_2400_ac_electric_level`
  - `sensor.solarflow_2400_ac_available_kwh`

---

## 🧪 Enthaltene Automationen

Name | Funktion
-----|---------
`Zendure Akku Automatik (V6)` | Smart-Charging, Entladeplanung, Prognose, Reserve-SOC
`Zendure – Max Ladeleistung übernehmen` | Sofortiges Anwenden des Lade-Sliders
`Zendure – Max Entladeleistung übernehmen` | Sofortiges Anwenden des Entlade-Sliders

---

## 📊 Dashboard
Ein fertiges Dashboard (mit Mushroom oder Standard-HA-Karten) ist enthalten, inklusive:

- SOC-Anzeige
- Lade/Entlade-Leistung
- Strompreis live
- Prognose-Auswertung
- Slider für Leistungsgrenzen
- Debug-Status

---

## ✅ Funktionen in Kürze
- Priorisiert PV-Überschuss
- Lädt automatisch bei tiefen Strompreisen
- Entlädt automatisch bei teuren Preisspitzen
- Dynamische Reserve, dynamische Leistung
- Schutz vor Tiefentladung
- Debug-Sensor erklärt Entscheidungen

---

# Zendure Home Assistant Steuerung – Automatik & Sommermodus  
Version: **v2.1**

Diese Anleitung beschreibt die komplette intelligente Zendure-Batteriesteuerung in Home Assistant:

- Dynamische Lade-/Entladeleistung
- PV-Überschussnutzung
- Preisabhängige Entladung im Automatik-Modus
- Sommermodus für maximale Autarkie
- **NEU: Notladeschutz unter 8 % (300 W)**
- Steuerung vollständig über Dashboard-Schieberegler
- 5-Minuten-Watchdog, damit das System nie „hängen bleibt“

---

## ✅ Benötigte Entitäten

### ### **Sensoren**
| Name | Zweck |
|------|-------|
`sensor.solarflow_2400_ac_electric_level` | State of Charge (SoC) in %  
`sensor.sb2_5_1vl_40_401_pv_power` | PV-Leistung  
`sensor.electricity_price_paul_schneider_strasse_39` | Aktueller Tibber-Preis  
`sensor.gesamtverbrauch` | Hausverbrauch  
`sensor.einspeisung` | PV-Überschuss  
`sensor.zendure_akku_steuerungsempfehlung` | Empfehlung der Preis-/Prognoselogik

---

## ✅ Eingaberegler (Helpers)

```yaml
input_select:
  zendure_betriebsmodus:
    name: Zendure Betriebsmodus
    options:
      - Automatik
      - Sommer
      - Manuell
    icon: mdi:battery-sync

input_number:
  zendure_max_ladeleistung:
    name: Max. Ladeleistung
    min: 50
    max: 2400
    step: 50
    unit_of_measurement: W
    icon: mdi:battery-charging

  zendure_max_entladeleistung:
    name: Max. Entladeleistung
    min: 50
    max: 2400
    step: 50
    unit_of_measurement: W
    icon: mdi:battery-arrow-down

  zendure_soc_reserve_min:
    name: SOC-Reserve (Entladegrenze)
    min: 0
    max: 30
    step: 1
    unit_of_measurement: '%'
    icon: mdi:battery-heart

  zendure_soc_ziel_max:
    name: SOC-Ziel (max Laden)
    min: 50
    max: 100
    step: 1
    unit_of_measurement: '%'
    icon: mdi:battery-high

  zendure_soc_notfall_min:
    name: Zendure SOC Notfallminimum
    min: 0
    max: 20
    step: 1
    unit_of_measurement: '%'
    icon: mdi:battery-alert
    initial: 8

  zendure_notladeleistung:
    name: Zendure Notladeleistung
    min: 50
    max: 1000
    step: 50
    unit_of_measurement: W
    icon: mdi:flash-alert
    initial: 300

---

## ❗ Hinweis
Diese Steuerung ersetzt keine werkseitigen Sicherheitsfunktionen des Akkus.  
Alle Eingriffe erfolgen über offizielle Home-Assistant-Schnittstellen.

---

Viel Spaß beim Optimieren – Beiträge & Verbesserungen sind willkommen!
