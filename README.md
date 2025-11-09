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
