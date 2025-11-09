# 📄 Zendure Home Assistant Steuerung – Vollständige Anleitung (Einsteigerfreundlich)
**Version 2.1 – Automatik, Sommermodus & Notladeschutz**

Diese Anleitung beschreibt eine komplette, praxiserprobte Home‑Assistant‑Steuerung für den **Zendure SolarFlow** Akku.

- ✅ PV‑Überschuss laden (mit 50 W Sicherheitspuffer)
- ✅ Laden bei sehr niedrigen/negativen Strompreisen
- ✅ Entladen bei hohen Preisen (Automatik)
- ✅ **Sommermodus**: maximale Autarkie (abends automatisch entladen)
- ✅ **Notladeschutz**: unter 8 % SoC automatisch mit 300 W nachladen
- ✅ Limits & SoC‑Ziele per Dashboard (Helfer) einstellbar
- ✅ 5‑Minuten‑Watchdog (verhindert „Einschlafen“ der Logik)

---

## Inhalt
1. Voraussetzungen & benötigte Entitäten
2. Helfer (input_select / input_number) – vollständiger YAML‑Code
3. Haupt‑Automation (V5) – vollständiger YAML‑Code
4. Dashboard‑Beispiele (Entities/Mushroom)
5. Wie die Logik arbeitet (einfach erklärt)
6. Fehlerbehebung (Troubleshooting)
7. Changelog zur Anleitung

---

## 1) Voraussetzungen & benötigte Entitäten

### Benötigte HA‑Entitäten (von Integrationen/Geräten)
| Typ | Entität | Beschreibung |
|-----|--------|--------------|
| sensor | `sensor.solarflow_2400_ac_electric_level` | Akkustand SoC in % |
| number | `number.solarflow_2400_ac_input_limit` | Eingestellte Ladeleistung (W) |
| number | `number.solarflow_2400_ac_output_limit` | Eingestellte Entladeleistung (W) |
| select | `select.solarflow_2400_ac_ac_mode` | Modus `input` (laden) / `output` (entladen) |
| sensor | `sensor.sb2_5_1vl_40_401_pv_power` | PV‑Leistung (W) |
| sensor | `sensor.einspeisung` | Momentane Einspeiseleistung (W) |
| sensor | `sensor.gesamtverbrauch` | Hausverbrauch (W) |
| sensor | `sensor.electricity_price_paul_schneider_strasse_39` | Aktueller Tibber‑Preis (€/kWh) |

> Optional empfohlen: Preisprognose 15‑min, Debug‑Sensoren.

---

## 2) Helfer anlegen (Dashboard‑Slider & Modus)
Du kannst die Helfer im UI (Einstellungen → Geräte & Dienste → **Helfer**) anlegen **oder** den YAML‑Block in deine `configuration.yaml` einfügen.

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
    name: Max Ladeleistung
    min: 50
    max: 2400
    step: 50
    unit_of_measurement: W
    icon: mdi:battery-charging

  zendure_max_entladeleistung:
    name: Max Entladeleistung
    min: 50
    max: 2400
    step: 50
    unit_of_measurement: W
    icon: mdi:battery-arrow-down

  zendure_soc_reserve_min:
    name: SOC Reserve (Entladegrenze)
    min: 0
    max: 30
    step: 1
    unit_of_measurement: '%'
    icon: mdi:battery-heart
    initial: 15

  zendure_soc_ziel_max:
    name: SOC Ziel (max. Ladung)
    min: 50
    max: 100
    step: 1
    unit_of_measurement: '%'
    icon: mdi:battery-high
    initial: 95

  zendure_soc_notfall_min:
    name: Notfall SOC (Tiefentladungsschutz)
    min: 0
    max: 20
    step: 1
    unit_of_measurement: '%'
    icon: mdi:battery-alert
    initial: 8

  zendure_notladeleistung:
    name: Notlade-Leistung
    min: 50
    max: 1000
    step: 50
    unit_of_measurement: W
    icon: mdi:flash-alert
    initial: 300
```

---

## 3) Haupt‑Automation (V5) – **kompletter YAML‑Code**

**Ort:** Einstellungen → Automationen → „Erstellen“ → **YAML‑Modus** wählen → **kompletten Block einfügen**

```yaml
alias: Zendure Akku Automatik (V5 mit Sommermodus & Notladeschutz)
description: Intelligente Akku-Steuerung (Sommer/Automatik + Notladung unter 8%)
mode: single

trigger:
  - platform: state
    entity_id:
      - sensor.zendure_akku_steuerungsempfehlung
      - sensor.sb2_5_1vl_40_401_pv_power
      - sensor.electricity_price_paul_schneider_strasse_39
      - sensor.gesamtverbrauch
      - sensor.einspeisung
      - input_select.zendure_betriebsmodus
  - platform: time_pattern
    minutes: "/5"  # Watchdog: prüft alle 5 Minuten

actions:
  - variables:
      soc: "{{ states('sensor.solarflow_2400_ac_electric_level') | float(0) }}"
      soc_min: "{{ states('input_number.zendure_soc_reserve_min') | float(15) }}"
      soc_max: "{{ states('input_number.zendure_soc_ziel_max') | float(95) }}"
      soc_notfall: "{{ states('input_number.zendure_soc_notfall_min') | float(8) }}"
      notlade: "{{ states('input_number.zendure_notladeleistung') | float(300) }}"
      price: "{{ states('sensor.electricity_price_paul_schneider_strasse_39') | float(0) }}"
      pv: "{{ states('sensor.sb2_5_1vl_40_401_pv_power') | float(0) }}"
      einspeisung: "{{ states('sensor.einspeisung') | float(0) }}"
      haus: "{{ states('sensor.gesamtverbrauch') | float(0) }}"
      netto: "{{ [haus - pv, 0] | max }}"
      max_charge: "{{ states('input_number.zendure_max_ladeleistung') | float(2000) }}"
      max_out: "{{ states('input_number.zendure_max_entladeleistung') | float(600) }}"
      modus: "{{ states('input_select.zendure_betriebsmodus') | default('Automatik') }}"
      sommer_pv_low: 100   # PV < 100 W -> Abend/Nacht
      sommer_cheap: 0.05   # <= 5 ct/kWh -> Netzladen erlaubt
      sommer_free: 0.00    # <= 0 ct/kWh -> Volllast laden

  - choose:

      # 1) NOTLADESCHUTZ – verhindert Tiefentladung (immer vorrangig)
      - conditions:
          - condition: template
            value_template: "{{ soc <= soc_notfall }}"
        sequence:
          - service: select.select_option
            target: { entity_id: select.solarflow_2400_ac_ac_mode }
            data: { option: input }
          - service: number.set_value
            target: { entity_id: number.solarflow_2400_ac_input_limit }
            data: { value: "{{ notlade }}" }
          - service: number.set_value
            target: { entity_id: number.solarflow_2400_ac_output_limit }
            data: { value: "0" }
          - stop: "true"

      # 2) SOMMER-MODUS – Autarkie priorisieren
      - conditions:
          - condition: template
            value_template: "{{ modus == 'Sommer' }}"
        sequence:

          # 2A) PV-Überschuss laden (mit 50 W Puffer)
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ pv > sommer_pv_low and soc < soc_max }}"
                sequence:
                  - service: select.select_option
                    target: { entity_id: select.solarflow_2400_ac_ac_mode }
                    data: { option: input }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_input_limit }
                    data:
                      value: "{{ [pv - 50, max_charge] | min | float(0) }}"

          # 2B) Sehr billiger Preis -> Netzladen erlaubt
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ price <= sommer_cheap and soc < soc_max }}"
                sequence:
                  - service: select.select_option
                    target: { entity_id: select.solarflow_2400_ac_ac_mode }
                    data: { option: input }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_input_limit }
                    data: { value: "{{ max_charge }}" }

          # 2C) Gratis / Negativpreis -> Volllast
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ price <= sommer_free and soc < soc_max }}"
                sequence:
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_input_limit }
                    data: { value: "{{ max_charge }}" }

          # 2D) Abend/Nacht (PV < 100 W) -> Entladen bis Reserve
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ pv < sommer_pv_low and soc > soc_min }}"
                sequence:
                  - service: select.select_option
                    target: { entity_id: select.solarflow_2400_ac_ac_mode }
                    data: { option: output }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_output_limit }
                    data: { value: "{{ [netto, max_out] | min | float(0) }}" }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_input_limit }
                    data: { value: "0" }

          - stop: "true"

      # 3) AUTOMATIK – Preisoptimiert
      - conditions:
          - condition: template
            value_template: "{{ modus == 'Automatik' }}"
        sequence:

          # 3A) PV-Überschuss laden (mit 50 W Puffer)
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ einspeisung > 0 and soc < soc_max }}"
                sequence:
                  - service: select.select_option
                    target: { entity_id: select.solarflow_2400_ac_ac_mode }
                    data: { option: input }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_input_limit }
                    data:
                      value: "{{ [einspeisung - 50, max_charge] | min | float(0) }}"

          # 3B) Teure Preise -> Entladen
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ price >= (states('input_number.zendure_schwelle_teuer') | float(0.40)) and soc > soc_min }}"
                sequence:
                  - service: select.select_option
                    target: { entity_id: select.solarflow_2400_ac_ac_mode }
                    data: { option: output }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_output_limit }
                    data: { value: "{{ [netto, max_out] | min | float(0) }}" }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_input_limit }
                    data: { value: "0" }

          # 3C) Standby -> Limits auf 0
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ einspeisung <= 0 and price < (states('input_number.zendure_schwelle_teuer') | float(0.40)) }}"
                sequence:
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_output_limit }
                    data: { value: "0" }
                  - service: number.set_value
                    target: { entity_id: number.solarflow_2400_ac_input_limit }
                    data: { value: "0" }
```

---

## 4) Dashboard‑Beispiele

### A) Kompakte **Entities‑Card** (Standard)
```yaml
type: entities
title: Zendure Einstellungen
show_header_toggle: false
entities:
  - input_select.zendure_betriebsmodus
  - input_number.zendure_max_ladeleistung
  - input_number.zendure_max_entladeleistung
  - input_number.zendure_soc_reserve_min
  - input_number.zendure_soc_ziel_max
  - input_number.zendure_soc_notfall_min
  - input_number.zendure_notladeleistung
```

### B) Mushroom (falls installiert)
```yaml
type: vertical-stack
cards:
  - type: custom:mushroom-entity-card
    entity: input_select.zendure_betriebsmodus
    name: Betriebsmodus
    icon: mdi:battery-sync
    primary_info: name
    secondary_info: state
  - type: entities
    title: Leistung & Sicherheit
    entities:
      - input_number.zendure_max_ladeleistung
      - input_number.zendure_max_entladeleistung
      - input_number.zendure_soc_reserve_min
      - input_number.zendure_soc_ziel_max
      - input_number.zendure_soc_notfall_min
      - input_number.zendure_notladeleistung
```

---

## 5) Wie die Logik arbeitet (einfach erklärt)

- **PV‑Überschuss**: Sobald `einspeisung > 0`, lädt der Akku. Ein **Puffer von 50 W** verhindert kurze Netzbezüge durch Verzögerungen.
- **Automatik (Preisoptimiert)**: Bei teuren Preisen wird entladen (bis Hausbedarf oder Max‑Output). Bei günstigen Preisen wird geladen (über Empfehlung/Prognose – optional).
- **Sommermodus (Autarkie)**: Tagsüber mit PV laden, abends/nachts (**PV < 100 W**) automatisch entladen – **ohne Preisschwelle**. Ziel: Netzbezug minimieren.
- **SoC‑Ziele**: `zendure_soc_reserve_min` verhindert Tiefentladung (Entladen stoppt). `zendure_soc_ziel_max` begrenzt das Laden.
- **Notladeschutz (neu)**: Fällt der SoC **≤ 8 %**, wird **immer** mit `zendure_notladeleistung` (Standard 300 W) geladen, bis die normale Reserve wieder überschritten wird – unabhängig von Preis/Modus.
- **Watchdog**: Ein Zeit‑Trigger alle 5 Minuten sorgt dafür, dass die Automation nie „einschläft“, selbst wenn Preis/PV sich nicht ändern.

---

## 6) Fehlerbehebung (Troubleshooting)

| Problem | Mögliche Ursache | Lösung |
|--------|-------------------|-------|
| Automation speichert nicht | Einrückung/Tabulatoren | Nur Leerzeichen, YAML korrekt einrücken |
| Akku lädt nicht trotz Sonne | PV‑Wert < 100 W, SoC ≥ Ziel | Schwellen prüfen (`sommer_pv_low`, `soc_max`) |
| Entlädt nicht bei „teuer“ | SoC ≤ Reserve, Preis unter Schwelle | `soc_min`/Preis‑Schwelle prüfen |
| Limits bleiben auf 0 | Modus nicht gesetzt | `select.solarflow_2400_ac_ac_mode` prüfen |
| Negative Preise werden nicht genutzt | `sommer_free` falsch | `sommer_free: 0.00` setzen |
| System „schläft“ | Keine Trigger aktiv | Watchdog ist aktiv, zusätzlich manuell triggern (Modus wechseln) |

---

## 7) Changelog (Anleitung)

**v2.1**
- Neu: **Notladeschutz** (SoC ≤ 8 %) lädt mit 300 W bis Reserve
- Sommermodus erweitert: PV‑Puffer, Abend‑Entladung, Netzladen bei ≤ 5 ct, Vollgas bei ≤ 0 ct
- 5‑Minuten‑Watchdog
- Vollständige, bereinigte YAML‑Automation (V5) in dieser Anleitung
