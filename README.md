# Samsung Washer & Fridge Card

> Lovelace cards voor Home Assistant, gebaseerd op [phrz/lg-washer-dryer-card](https://github.com/phrz/lg-washer-dryer-card) — aangepast voor **Samsung SmartThings** apparaten.

![Home Assistant](https://img.shields.io/badge/Home%20Assistant-compatible-blue?logo=home-assistant)
![HACS](https://img.shields.io/badge/HACS-manual-orange)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Inhoud

| Card | Bestand | Apparaat |
|---|---|---|
| Wasmachine | `samsung-washer-card.yaml` | Samsung wasmachine via SmartThings |
| Washer-dryer combo | `samsung-washercombo-card.yaml` | Samsung combo (wassen + drogen) via SmartThings |
| Koelvriescombinatie | `samsung-fridge-card.yaml` | Samsung koelvriescombinatie via SmartThings |

---

## Vereisten

### Integraties
- [SmartThings](https://www.home-assistant.io/integrations/smartthings/) — officiële HA integratie
- Samsung account gekoppeld aan SmartThings

### Frontend resources (HACS of handmatig)
- **7segment.css** — voor het segment-display lettertype op de cards  
  Bron: originele [lg-washer-dryer-card](https://github.com/phrz/lg-washer-dryer-card/tree/main/config/www)

  Toevoegen via Dashboard → drie puntjes → **Bronnen beheren** → Toevoegen:
  ```
  /local/7segment.css   (type: Stylesheet)
  ```

### Achtergrondafbeelding
De cards gebruiken een achtergrondafbeelding als basis voor de picture-elements overlay.  
Gebruik tijdelijk de LG originele achtergrond, of vervang door een eigen Samsung-stijl afbeelding:

```
/config/www/hass-washer-card-bg.png
```

---

## Installatie

1. Kopieer de gewenste `.yaml` bestanden naar je Home Assistant configuratiemap
2. Zorg dat de frontend resource (`7segment.css`) beschikbaar is onder `/local/`
3. Voeg de card toe via de Lovelace YAML-editor:
   - Dashboard → Bewerken → drie puntjes → **Raw configuratie editor**
   - Of voeg toe als losse card via **Code-editor** in de kaartenkiezer

---

## Wasmachine card

### Entiteiten

| Entiteit | Rol |
|---|---|
| `sensor.was_machinestatus` | Machinestatus (`stop` / `run` / `pause`) |
| `sensor.was_taakstatus` | Taakstatus (`wash` / `rinse` / `spin` / `none` / `finished`) |
| `sensor.was_resterende_tijd` | Resterende tijd (bijv. `0:32`) |
| `binary_sensor.was_afstandsbediening` | Wifi / afstandsbediening actief |
| `binary_sensor.was_child_lock` | Kinderbeveiliging |
| `select.was_water_temperature` | Ingestelde watertemperatuur |
| `select.was_centrifuge_snelheid` | Ingestelde centrifugesnelheid |
| `sensor.was_progressie` | Voortgang in % |

### States wasmachine

De card reageert op de volgende SmartThings states:

| `sensor.was_taakstatus` | Weergave |
|---|---|
| `wash` | Was-icoon actief |
| `rinse` | Spoelen-icoon actief |
| `spin` | Centrifuge-icoon actief |
| `none` / `finished` | Alle iconen inactief |

---

## Washer-dryer combo card

### Entiteiten
Zelfde als de wasmachine card — geen extra entiteiten nodig. De card detecteert automatisch was- vs droog-fases via `sensor.was_taakstatus`.

### Ondersteunde taakstatussen

| State | Fase | Icoon |
|---|---|---|
| `pre_wash` | Voorwas | 💧− |
| `wash` / `ai_wash` | Wassen | 💧 |
| `rinse` / `ai_rinse` | Spoelen | 🔄 |
| `spin` / `ai_spin` | Centrifuge | ↻ |
| `drying` | Drogen | 🌀 oranje |
| `cooling` | Afkoelen | ❄️ blauw |
| `wrinkle_prevent` | Anti-kreuk | 🔶 oranje |
| `finish` | Klaar | ✅ groen |
| `none` | Standby | alle iconen grijs |

---

## Koelvriescombinatie card

### Entiteiten

| Entiteit | Rol |
|---|---|
| `binary_sensor.koelvriescombinatie_fridge_door` | Koelkastdeur open/dicht |
| `binary_sensor.koelvriescombinatie_freezer_door` | Vriezerdeur open/dicht |
| `number.koelvriescombinatie_koelkasttemperatuur` | Ingestelde koelkasttemperatuur (°C) |
| `number.koelvriescombinatie_freezer_temperature` | Ingestelde vriezertemperatuur (°C) |
| `sensor.koelvriescombinatie_vermogen` | Huidig vermogen (W) |
| `sensor.koelvriescombinatie_energie` | Totaal energieverbruik (kWh) |
| `switch.koelvriescombinatie_power_cool` | Power Cool (snelkoelen) |
| `switch.koelvriescombinatie_power_freeze` | Power Freeze (snelvriezen) |

### Bekende issues — temperatuur

De `sensor.*_temperature` entiteiten van SmartThings geven soms Fahrenheit terug terwijl de eenheid °C aangeeft. Voeg onderstaande template sensors toe als workaround:

```yaml
template:
  - sensor:
      - name: "Koelkast temperatuur °C"
        unique_id: koelkast_temp_celsius
        unit_of_measurement: "°C"
        device_class: temperature
        state: >
          {{ ((states('sensor.koelvriescombinatie_koelkasttemperatuur') | float - 32) * 5/9) | round(1) }}

      - name: "Vriezer temperatuur °C"
        unique_id: vriezer_temp_celsius
        unit_of_measurement: "°C"
        device_class: temperature
        state: >
          {{ ((states('sensor.koelvriescombinatie_freezer_temperature') | float - 32) * 5/9) | round(1) }}
```

Vervang daarna in de card de `number.*` entiteiten door `sensor.koelkast_temp_celsius` en `sensor.vriezer_temp_celsius`.

---

## Aanpassen aan jouw entiteitnamen

Alle entiteit-ID's in de YAML-bestanden zijn gebaseerd op één specifieke setup. Zoek en vervang de prefixen als jouw apparaten anders heten:

| Prefix in deze repo | Vervang door |
|---|---|
| `was_` | jouw wasmachine prefix |
| `koelvriescombinatie_` | jouw koelkast prefix |

---

## Credits

Gebaseerd op het uitstekende werk van [@phrz](https://github.com/phrz):  
[phrz/lg-washer-dryer-card](https://github.com/phrz/lg-washer-dryer-card) — originele LG ThinQ Lovelace cards

---

## Licentie

MIT — vrij te gebruiken, aanpassen en delen.
