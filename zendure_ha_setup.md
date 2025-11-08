# Zendure Home Assistant – Komplettanleitung

Diese Anleitung bündelt alle aktuellen Bausteine deiner Zendure-Steuerung in Home Assistant:
- Helfer (input_number)
- Template-Sensoren (Debug & Prognose)
- Vollautomatik (V4 – mit Prognose & dynamischer Mindestenergie)
- Sofort-Übernahme der Slider (2 Mini-Automationen)
- Kompaktes Mushroom-Dashboard (Komfortkarte)
- Hinweise & Fehlerbehebung

> **Voraussetzungen**
> - Deine Zendure-Integration liefert folgende Entitäten (Beispiele):  
>   `select.solarflow_2400_ac_ac_mode`, `number.solarflow_2400_ac_input_limit`, `number.solarflow_2400_ac_output_limit`,  
>   `sensor.solarflow_2400_ac_electric_level`, `sensor.solarflow_2400_ac_available_kwh`,  
>   `sensor.sb2_5_1vl_40_401_pv_power`, `sensor.gesamtverbrauch`, `sensor.einspeisung`,  
>   `sensor.electricity_price_paul_schneider_strasse_39`, `sensor.strompreis_prognose_15min_paul_schneider_strasse_39`  
> - Der Sensor `sensor.zendure_akku_steuerungsempfehlung` existiert (wird als Trigger genutzt).

---

## 1) Helfer (input_number)

```yaml
input_number:
  zendure_max_ladeleistung:
    name: Max Ladeleistung
    min: 0
    max: 2400
    step: 50
    unit_of_measurement: W
    mode: slider

  zendure_max_entladeleistung:
    name: Max Entladeleistung
    min: 0
    max: 2400
    step: 50
    unit_of_measurement: W
    mode: slider

  zendure_soc_reserve_min:
    name: SoC-Reserve (Min)
    unit_of_measurement: '%'
    min: 0
    max: 60
    step: 1
    mode: slider
    initial: 20

  zendure_soc_ziel_max:
    name: SoC-Ziel (Max)
    unit_of_measurement: '%'
    min: 50
    max: 100
    step: 1
    mode: slider
    initial: 95

  zendure_schwelle_teuer:
    name: Preisgrenze Teuer
    min: 0.10
    max: 1.00
    step: 0.01
    unit_of_measurement: €/kWh
    mode: slider
    initial: 0.40

  zendure_schwelle_extrem:
    name: Preisgrenze Extrem teuer
    min: 0.10
    max: 1.00
    step: 0.01
    unit_of_measurement: €/kWh
    mode: slider
    initial: 0.50
```

---

## 2) Template-Sensoren

### 2.1 Entlade-Debug
```yaml
template:
  - sensor:
      - name: Zendure Entlade Debug
        state: >
          {% set soc = states('sensor.solarflow_2400_ac_electric_level')|float %}
          {% set available = states('sensor.solarflow_2400_ac_available_kwh')|float %}
          {% set price = states('sensor.electricity_price_paul_schneider_strasse_39')|float %}
          {% set gepl = states('sensor.zendure_geplante_entladeleistung')|float %}
          {% set needed = (gepl/1000) * 1.5 %}
          {% set exp = states('input_number.zendure_schwelle_teuer')|float(0.40) %}
          {% set vexp = states('input_number.zendure_schwelle_extrem')|float(0.50) %}

          Preis aktuell: {{price}} €/kWh
          Verfügbare Energie: {{available}} kWh
          Benötigt für smarte Entladung: {{needed|round(2)}} kWh

          {% if price >= vexp %}
            Extrem teuer → maximale Entladung sinnvoll
          {% elif price >= exp %}
            Teuer → smarte Entladung möglich
          {% else %}
            Preis normal
          {% endif %}

          {% if available < needed %}
            Akku zu leer für geplante Entladung → vorher Laden
          {% else %}
            Akku ausreichend geladen für geplante Entladung
          {% endif %}
```

### 2.2 Nächstes teures Zeitfenster (Prognosebewertung)
```yaml
template:
  - sensor:
      - name: Zendure Nächstes Teures Zeitfenster
        state: >
          {% set today = state_attr('sensor.strompreis_prognose_15min_paul_schneider_strasse_39','today') %}
          {% set tomo  = state_attr('sensor.strompreis_prognose_15min_paul_schneider_strasse_39','tomorrow') %}
          {% set exp = states('input_number.zendure_schwelle_teuer')|float(0.40) %}
          {% set values = [] %}
          {% if today %}
            {% for step in today %}{% set values = values + [ step.total ] %}{% endfor %}
          {% endif %}
          {% if tomo %}
            {% for step in tomo %}{% set values = values + [ step.total ] %}{% endfor %}
          {% endif %}
          {% if values | select('>=', exp) | list | count > 0 %}
            teuer
          {% else %}
            günstig
          {% endif %}
```

---

## 3) Haupt-Automation (V4 – Prognose & dynamische Mindestenergie)

```yaml
alias: Zendure Akku Automatik (V4 – Prognose, dynamische Mindestenergie, FIX)
description: Vollautomatische Optimierung von Laden/Entladen mit Preisprognose
mode: single

trigger:
  - platform: state
    entity_id: sensor.zendure_akku_steuerungsempfehlung

variables:
  soc: "{{ states('sensor.solarflow_2400_ac_electric_level') | float }}"
  soc_min: "{{ states('input_number.zendure_soc_reserve_min') | float }}"
  soc_max: "{{ states('input_number.zendure_soc_ziel_max') | float }}"
  price: "{{ states('sensor.electricity_price_paul_schneider_strasse_39') | float }}"
  expensive: "{{ states('input_number.zendure_schwelle_teuer') | float(0.40) }}"
  very_expensive: "{{ states('input_number.zendure_schwelle_extrem') | float(0.50) }}"
  available_kwh: "{{ states('sensor.solarflow_2400_ac_available_kwh') | float }}"
  planned_w: "{{ states('sensor.zendure_geplante_entladeleistung') | float }}"
  needed_kwh: "{{ (planned_w / 1000) * 1.5 }}"
  prognose: "{{ states('sensor.zendure_naechstes_teures_zeitfenster') }}"

condition: []

action:
  - choose:

      # 1) ENTLADEEMPFEHLUNG – AKKU ZU LEER → VOR-LADEN
      - conditions:
          - condition: template
            value_template: >
              {{ trigger.to_state.state == 'entladen'
                 and soc > soc_min
                 and available_kwh < needed_kwh }}
        sequence:
          - service: select.select_option
            target:
              entity_id: select.solarflow_2400_ac_ac_mode
            data:
              option: input
          - service: number.set_value
            target:
              entity_id: number.solarflow_2400_ac_input_limit
            data:
              value: "{{ states('input_number.zendure_max_ladeleistung') | float }}"
          - service: number.set_value
            target:
              entity_id: number.solarflow_2400_ac_output_limit
            data:
              value: "0"

      # 2) ENTLADEEMPFEHLUNG – GENUG ENERGIE → ENTLADEN
      - conditions:
          - condition: template
            value_template: >
              {{ trigger.to_state.state == 'entladen'
                 and soc > soc_min
                 and available_kwh >= needed_kwh }}
        sequence:
          - service: select.select_option
            target:
              entity_id: select.solarflow_2400_ac_ac_mode
            data:
              option: output

          - variables:
              haus: "{{ states('sensor.gesamtverbrauch') | float }}"
              pv: "{{ states('sensor.sb2_5_1vl_40_401_pv_power') | float }}"
              netto: "{{ [haus - pv, 0] | max }}"
              max_out: "{{ states('input_number.zendure_max_entladeleistung') | float }}"

          - choose:
              # 2a) EXTREM TEUER → maximale Entladung
              - conditions:
                  - condition: template
                    value_template: "{{ price >= very_expensive }}"
                sequence:
                  - service: number.set_value
                    target:
                      entity_id: number.solarflow_2400_ac_output_limit
                    data:
                      value: "{{ [max_out, netto] | min | round(0) }}"
                  - service: number.set_value
                    target:
                      entity_id: number.solarflow_2400_ac_input_limit
                    data:
                      value: "0"

              # 2b) TEUER → smarte Entladung
              - conditions:
                  - condition: template
                    value_template: "{{ price >= expensive }}"
                sequence:
                  - variables:
                      smart: "{{ [planned_w, max_out, netto] | min }}"
                  - service: number.set_value
                    target:
                      entity_id: number.solarflow_2400_ac_output_limit
                    data:
                      value: "{{ [smart, 0] | max | round(0) }}"
                  - service: number.set_value
                    target:
                      entity_id: number.solarflow_2400_ac_input_limit
                    data:
                      value: "0"

            # 2c) Preis doch nicht teuer → stoppen
            default:
              - service: number.set_value
                target:
                  entity_id: number.solarflow_2400_ac_output_limit
                data:
                  value: "0"
              - service: number.set_value
                target:
                  entity_id: number.solarflow_2400_ac_input_limit
                data:
                  value: "0"

      # 3) LADEN – Empfehlung ODER Voraus-Laden für teures Fenster
      - conditions:
          - condition: template
            value_template: >
              {{ (trigger.to_state.state in ['laden','billig_laden'] and soc < soc_max)
                 or (prognose == 'teuer' and available_kwh < needed_kwh and soc < soc_max) }}
        sequence:
          - service: select.select_option
            target:
              entity_id: select.solarflow_2400_ac_ac_mode
            data:
              option: input

          - variables:
              max_charge: "{{ states('input_number.zendure_max_ladeleistung') | float }}"
              einspeisung: "{{ states('sensor.einspeisung') | float }}"
              ueberschuss: "{{ [einspeisung, 0] | max }}"
              taper: >
                {% set span = 5 %}
                {% set rem = soc_max - soc %}
                {% if rem <= 0 %} 0
                {% elif rem >= span %} 1
                {% else %} {{ rem / span }}
                {% endif %}
              limit_base: >
                {% if trigger.to_state.state == 'billig_laden' %}
                  {{ max_charge }}
                {% else %}
                  {{ [ueberschuss, max_charge] | min }}
                {% endif %}
              charge_limit: "{{ (limit_base * taper) | float }}"

          - service: number.set_value
            target:
              entity_id: number.solarflow_2400_ac_input_limit
            data:
              value: "{{ [charge_limit, 0] | max | round(0) }}"
          - service: number.set_value
            target:
              entity_id: number.solarflow_2400_ac_output_limit
            data:
              value: "0"

      # 4) STANDBY / STOP
      - conditions:
          - condition: template
            value_template: >
              {{ trigger.to_state.state == 'standby'
                 or (trigger.to_state.state in ['laden','billig_laden'] and soc >= soc_max)
                 or (trigger.to_state.state == 'entladen' and soc <= soc_min) }}
        sequence:
          - service: number.set_value
            target:
              entity_id: number.solarflow_2400_ac_input_limit
            data:
              value: "0"
          - service: number.set_value
            target:
              entity_id: number.solarflow_2400_ac_output_limit
            data:
              value: "0"
```

---

## 4) Sofort-Übernahme der Slider (optional, empfohlen)

```yaml
alias: Zendure - Max Entladeleistung übernehmen
trigger:
  - platform: state
    entity_id: input_number.zendure_max_entladeleistung
action:
  - service: number.set_value
    target:
      entity_id: number.solarflow_2400_ac_output_limit
    data:
      value: "{{ states('input_number.zendure_max_entladeleistung') | float }}"
mode: restart
```

```yaml
alias: Zendure - Max Ladeleistung übernehmen
trigger:
  - platform: state
    entity_id: input_number.zendure_max_ladeleistung
action:
  - service: number.set_value
    target:
      entity_id: number.solarflow_2400_ac_input_limit
    data:
      value: "{{ states('input_number.zendure_max_ladeleistung') | float }}"
mode: restart
```

---

## 5) Mushroom – kompakte Komfortkarte

```yaml
type: vertical-stack
title: ⚡ Zendure Leistungskontrolle
cards:

  - type: custom:mushroom-chips-card
    chips:
      - type: entity
        entity: input_number.zendure_max_ladeleistung
        name: Max Laden (W)
        icon: mdi:battery-arrow-up
      - type: entity
        entity: input_number.zendure_max_entladeleistung
        name: Max Entladen (W)
        icon: mdi:battery-arrow-down

  - type: horizontal-stack
    cards:
      - type: custom:mushroom-entity-card
        entity: number.solarflow_2400_ac_input_limit
        name: Lade-Limit aktiv
        icon: mdi:arrow-collapse-up
        layout: vertical
        primary_info: state
        secondary_info: name
        icon_color: blue

      - type: custom:mushroom-entity-card
        entity: number.solarflow_2400_ac_output_limit
        name: Entlade-Limit aktiv
        icon: mdi:arrow-collapse-down
        layout: vertical
        primary_info: state
        secondary_info: name
        icon_color: amber

  - type: horizontal-stack
    cards:
      - type: custom:mushroom-entity-card
        entity: sensor.solarflow_2400_ac_input_power
        name: lädt gerade (W)
        icon: mdi:lightning-bolt
        icon_color: blue
        layout: vertical
      - type: custom:mushroom-entity-card
        entity: sensor.solarflow_2400_ac_output_home_power
        name: entlädt gerade (W)
        icon: mdi:lightning-bolt
        icon_color: amber
        layout: vertical
```

---

## 6) Hinweise & Fehlerbehebung

- Wenn das Gerät in HA nicht über 600 W hinausgeht, **prüfe das Geräte-Limit in der Zendure-App** (Input/Output auf gewünschten Maximalwert stellen, z. B. 2400 W).
- Die **Automation** orientiert sich **immer** an deinen `input_number`-Grenzen – nicht an der Anzeige der Geräteregler.
- Für die Prognose wird `sensor.strompreis_prognose_15min_paul_schneider_strasse_39` genutzt. Achte darauf, dass er Daten für **heute und morgen** liefert.
- Typische YAML-Fehler beim Speichern: Einrückungen, `choose:`/`default:`-Ebene, Tabs statt Leerzeichen.
- Teste Entscheidungen bequem über den Debug-Sensor `sensor.zendure_entlade_debug`.

Fertig! 💚
