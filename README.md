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
- Haupt-Automation (V4)
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
`Zendure Akku Automatik (V4)` | Smart-Charging, Entladeplanung, Prognose, Reserve-SOC
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

## ❗ Hinweis
Diese Steuerung ersetzt keine werkseitigen Sicherheitsfunktionen des Akkus.  
Alle Eingriffe erfolgen über offizielle Home-Assistant-Schnittstellen.

---

Viel Spaß beim Optimieren – Beiträge & Verbesserungen sind willkommen!
