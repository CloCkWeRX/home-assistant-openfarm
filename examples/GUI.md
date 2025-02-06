# Example GUI

This is a very rough example on what I have set up to create the HA GUI.

![Example](../images/openplantbook.gif)


## Helpers

We need some helpers to save the search results and trigger the automatons

### input_text
```
openplantbook_search:
  name: Search OpenPlantbook
```

### input_select
```
openplantbook_searchresults:
  name: Openplantbook Search Results
  options:
    - "Search first"
```
### input_button
```
openplantbook_clear_cache:
  name: Clear Openplantbook Cache
```

## Automations

These automations trigger when the different helpers are modified.

Initiate a search when the input_text helper is modified

```
alias: Search Openfarm
trigger:
  - platform: state
    entity_id: input_textopenfarm_search
action:
  - service: openfarm.search
    data:
      alias: "{{ states('input_text.openfarm_search') }}"

```

Populate the input_select when a search result is ready

```
alias: Populate Openfarm Dropdown
trigger:
  - platform: state
    entity_id: openfarm.search_result
action:
  - service: input_select.set_options
    data:
      entity_id: input_select.openfarm_searchresults
      options: |
        {% if states('openfarm.search_result') | int(default=0) > 0 %}
          {{ states.openfarm.search_result.attributes | list }}
        {% else %}
          [ "No plants found"]
        {% endif %}

```

Get details from Openfarm when an option in the dropdown is selected

```
alias: Get Info From Openfarm
trigger:
  - platform: state
    entity_id: input_select.openplantbook_searchresults
action:
  - service: openfarm.get
    data:
      species: "{{ states('input_select.openplantbook_searchresults') }}"

```

Clear the cache when the button is pressed

```

alias: Clear Openfarm cache
trigger:
  - platform: state
    entity_id:
      - input_button.openfarm_clear_cache
action:
  - service: openfarm.clean_cache
    data:
      hours: 0
```



## Lovelace

I use two cards.  One for the search and search results, and one to display the info about a single plant.

```
type: entities
title: Search Openfarm
entities:
  - entity: input_text.openfarm_search
  - entity: openfarm.search_result
  - entity: input_select.openfarm_searchresults
  - entity: input_button.openfarm_clear_cache
```

```
type: markdown
title: Plant info
content: |

  {% set plant = "openfarm." + states('input_select.openplantbook_searchresults') | replace(" ", "_") | replace("'", "") | replace(".", "") %}
  {% if states(plant) == "unknown" %}
  # Search for a plant
  {% else %}
  # {{ state_attr(plant, 'display_pid') }}
  {% set min_dli = ((state_attr(plant, 'min_light_mmol') | float(default=0) * 0.0036)) | round(0, default=0) %}
  {% set max_dli = ((state_attr(plant, 'max_light_mmol') | float(default=0) * 0.0036)) | round(0, default=0) %}

  ![Image]({{ state_attr(plant, 'image_url') | urlencode }})

  _{{ state_attr(plant, 'pid') }}_

  ## Thresholds

  |                      | Min                                       |    | Max                                       |      |
  |----------------------|------------------------------------------:|----|------------------------------------------:|------|
  | Moisture             | {{ state_attr(plant, 'min_soil_moist') }} |    | {{ state_attr(plant, 'max_soil_moist') }} |%     |
  | Conductitivty        | {{ state_attr(plant, 'min_soil_ec') }}    |    | {{ state_attr(plant, 'max_soil_ec') }}    |μS/cm |
  | Temperature          | {{ state_attr(plant, 'min_temp') }}       |    | {{ state_attr(plant, 'max_temp') }}       |°C    |
  | Humidity             | {{ state_attr(plant, 'min_env_humid') }}  |    | {{ state_attr(plant, 'max_env_humid') }}  |%     |
  | Illumination         | {{ state_attr(plant, 'min_light_lux') }}  |    | {{ state_attr(plant, 'max_light_lux') }}  |lx    |
  | Daily Light Integral | {{ min_dli }}                             |    | {{ max_dli }}                             |mol/d⋅m²|
  {% endif %}
```
