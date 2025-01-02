# Smart Heating Dashboard Sensors

Below is a curated set of **sensor** definitions that **directly power** the **Heating Control** dashboard you shared. These include:
* **History Stats** sensors (to track daily heating time per room)
* **Template** sensors (to calculate costs, total heating hours, and active heating rooms)

Also included is the **image** demonstrating the resulting dashboard UI.

---

## Dashboard Screenshot

![Smart Heating Dashboard Screenshot](https://user-images.githubusercontent.com/youruser/yourrepo/yourimage.jpg)


---

## Relevant Sensor Configuration

```yaml
#===============================
# HISTORY_STATS Sensors
# (Tracks how many hours each room's HVAC is actively heating)
#===============================
sensor:

  - platform: history_stats
    name: ethan_heating_time
    entity_id: sensor.ethan_hvac_action
    state: 'heating'
    type: time
    start: '{{ now().replace(hour=0, minute=0, second=0) }}'
    end: '{{ now() }}'

  - platform: history_stats
    name: jacob_heating_time
    entity_id: sensor.jacob_hvac_action
    state: 'heating'
    type: time
    start: '{{ now().replace(hour=0, minute=0, second=0) }}'
    end: '{{ now() }}'

  - platform: history_stats
    name: kitchen_heating_time
    entity_id: sensor.kitchen_hvac_action
    state: 'heating'
    type: time
    start: '{{ now().replace(hour=0, minute=0, second=0) }}'
    end: '{{ now() }}'

  - platform: history_stats
    name: living_room_heating_time
    entity_id: sensor.living_room_hvac_action
    state: 'heating'
    type: time
    start: '{{ now().replace(hour=0, minute=0, second=0) }}'
    end: '{{ now() }}'

  - platform: history_stats
    name: master_bedroom_heating_time
    entity_id: sensor.master_bedroom_hvac_action
    state: 'heating'
    type: time
    start: '{{ now().replace(hour=0, minute=0, second=0) }}'
    end: '{{ now() }}'

  - platform: history_stats
    name: landing_heating_time
    entity_id: sensor.landing_hvac_action
    state: 'heating'
    type: time
    start: '{{ now().replace(hour=0, minute=0, second=0) }}'
    end: '{{ now() }}'


#===============================
# TEMPLATE Sensors
# (HVAC action, Active rooms, and cost calculations)
#===============================
template:
  - sensor:
      # Each room's hvac_action (used by history_stats above)
      - name: ethan_hvac_action 
        unique_id: ethan_hvac_action
        state: "{{ state_attr('climate.ethan', 'hvac_action') }}"

      - name: jacob_hvac_action
        unique_id: jacob_hvac_action
        state: "{{ state_attr('climate.jacob', 'hvac_action') }}"

      - name: kitchen_hvac_action
        unique_id: kitchen_hvac_action
        state: "{{ state_attr('climate.kitchen', 'hvac_action') }}"

      - name: living_room_hvac_action
        unique_id: living_room_hvac_action
        state: "{{ state_attr('climate.living_room', 'hvac_action') }}"

      - name: master_bedroom_hvac_action
        unique_id: master_bedroom_hvac_action
        state: "{{ state_attr('climate.master_bedroom', 'hvac_action') }}"

      - name: landing_hvac_action
        unique_id: landing_hvac_action
        state: "{{ state_attr('climate.landing', 'hvac_action') }}"


  #===============================
  # Additional template sensors grouped for clarity
  #===============================
  - sensor:
      # How many rooms are actively heating
      - name: active_heating_rooms_count
        unique_id: active_heating_rooms_count
        unit_of_measurement: "rooms"
        state: >
          {% set total = 0 %}
          {% for room in ['ethan', 'jacob', 'kitchen', 'living_room', 'master_bedroom', 'landing'] %}
            {% if states('sensor.' + room + '_heating_time') | float(default=0) > 0 %}
              {% set total = total + 1 %}
            {% endif %}
          {% endfor %}
          {{ total if total > 0 else 1 }}


      # Gas cost per active room
      - name: gas_cost_per_active_room
        unique_id: gas_cost_per_active_room
        unit_of_measurement: "GBP"
        state: >
          {% set active_rooms = namespace(count=0) %}
          {% for room in ['sensor.ethan_hvac_action', 'sensor.jacob_hvac_action', 
                          'sensor.kitchen_hvac_action', 'sensor.living_room_hvac_action', 
                          'sensor.master_bedroom_hvac_action', 'sensor.landing_hvac_action'] %}
            {% if states(room) == 'heating' %}
              {% set active_rooms.count = active_rooms.count + 1 %}
            {% endif %}
          {% endfor %}
          {% set gas_cost = states('sensor.octopus_energy_gas__current_accumulative_cost') | float(default=0) %}
          {{ (gas_cost / (active_rooms.count if active_rooms.count > 0 else 1)) | round(2) }}


      # Real-time cost per room
      - name: ethan_room_cost
        unit_of_measurement: "GBP"
        state: >
          {% if states('climate.ethan') == 'heat' and state_attr('climate.ethan', 'hvac_action') != 'idle' %}
            {% set gas_cost = states('sensor.octopus_energy_gas_current_accumulative_cost') | float(default=0) %}
            {% set active_rooms = states('sensor.active_heating_rooms_count') | int(default=1) %}
            {{ (gas_cost / active_rooms) | round(3) if gas_cost > 0 else 0 }}
          {% else %}
            0
          {% endif %}

      - name: jacob_room_cost
        unit_of_measurement: "GBP"
        state: >
          {% if is_state('climate.jacob', 'heat') %}
            {{ states('sensor.gas_cost_per_active_room') | float }}
          {% else %}
            0
          {% endif %}

      - name: kitchen_cost
        unit_of_measurement: "GBP"
        state: >
          {% if is_state('climate.kitchen', 'heat') %}
            {{ states('sensor.gas_cost_per_active_room') | float }}
          {% else %}
            0
          {% endif %}

      - name: living_room_cost
        unit_of_measurement: "GBP"
        state: >
          {% if is_state('climate.living_room', 'heat') %}
            {{ states('sensor.gas_cost_per_active_room') | float }}
          {% else %}
            0
          {% endif %}

      - name: master_bedroom_cost
        unit_of_measurement: "GBP"
        state: >
          {% if is_state('climate.master_bedroom', 'heat') %}
            {{ states('sensor.gas_cost_per_active_room') | float }}
          {% else %}
            0
          {% endif %}

      - name: landing_cost
        unit_of_measurement: "GBP"
        state: >
          {% if is_state('climate.landing', 'heat') %}
            {{ states('sensor.gas_cost_per_active_room') | float }}
          {% else %}
            0
          {% endif %}


      #===============================
      # Daily Cost Calculations
      #===============================
      - name: ethan_room_daily_cost
        unique_id: ethan_room_daily_cost
        unit_of_measurement: "GBP"
        state: >
          {% set room_hours = states('sensor.ethan_heating_time') | float(default=0) %}
          {% set total = states('sensor.total_heating_hours') | float(default=1) %}
          {% set gas_cost = states('sensor.octopus_energy_gas__current_accumulative_cost') | float(default=0) %}
          {{ ((gas_cost * room_hours) / total) | round(2) }}
        attributes:
          room_hours: "{{ states('sensor.ethan_heating_time') }}"
          total_hours: "{{ states('sensor.total_heating_hours') }}"
          gas_cost: "{{ states('sensor.octopus_energy_gas__current_accumulative_cost') }}"
          percentage: "{{ ((states('sensor.ethan_heating_time') | float(default=0) / states('sensor.total_heating_hours') | float(default=1)) * 100) | round(1) }}%"

      - name: jacob_room_daily_cost
        unique_id: jacob_room_daily_cost
        unit_of_measurement: "GBP"
        state: >
          {% set room_hours = states('sensor.jacob_heating_time') | float(default=0) %}
          {% set total = states('sensor.total_heating_hours') | float(default=1) %}
          {% set gas_cost = states('sensor.octopus_energy_gas__current_accumulative_cost') | float(default=0) %}
          {{ ((gas_cost * room_hours) / total) | round(2) if room_hours > 0 else 0 }}
        attributes:
          room_hours: "{{ states('sensor.jacob_heating_time') }}"
          total_hours: "{{ states('sensor.total_heating_hours') }}"
          gas_cost: "{{ states('sensor.octopus_energy_gas__current_accumulative_cost') }}"
          percentage: "{{ ((states('sensor.jacob_heating_time') | float(default=0) / states('sensor.total_heating_hours') | float(default=1)) * 100) | round(1) }}%"

      - name: kitchen_daily_cost
        unique_id: kitchen_daily_cost
        unit_of_measurement: "GBP"
        state: >
          {% set room_hours = states('sensor.kitchen_heating_time') | float(default=0) %}
          {% set total = states('sensor.total_heating_hours') | float(default=1) %}
          {% set gas_cost = states('sensor.octopus_energy_gas__current_accumulative_cost') | float(default=0) %}
          {{ ((gas_cost * room_hours) / total) | round(2) if room_hours > 0 else 0 }}
        attributes:
          room_hours: "{{ states('sensor.kitchen_heating_time') }}"
          total_hours: "{{ states('sensor.total_heating_hours') }}"
          gas_cost: "{{ states('sensor.octopus_energy_gas__current_accumulative_cost') }}"
          percentage: "{{ ((states('sensor.kitchen_heating_time') | float(default=0) / states('sensor.total_heating_hours') | float(default=1)) * 100) | round(1) }}%"

      - name: living_room_daily_cost
        unique_id: living_room_daily_cost
        unit_of_measurement: "GBP"
        state: >
          {% set room_hours = states('sensor.living_room_heating_time') | float(default=0) %}
          {% set total = states('sensor.total_heating_hours') | float(default=1) %}
          {% set gas_cost = states('sensor.octopus_energy_gas__current_accumulative_cost') | float(default=0) %}
          {{ ((gas_cost * room_hours) / total) | round(2) if room_hours > 0 else 0 }}
        attributes:
          room_hours: "{{ states('sensor.living_room_heating_time') }}"
          total_hours: "{{ states('sensor.total_heating_hours') }}"
          gas_cost: "{{ states('sensor.octopus_energy_gas__current_accumulative_cost') }}"
          percentage: "{{ ((states('sensor.living_room_heating_time') | float(default=0) / states('sensor.total_heating_hours') | float(default=1)) * 100) | round(1) }}%"

      - name: master_bedroom_daily_cost
        unique_id: master_bedroom_daily_cost
        unit_of_measurement: "GBP"
        state: >
          {% set room_hours = states('sensor.master_bedroom_heating_time') | float(default=0) %}
          {% set total = states('sensor.total_heating_hours') | float(default=1) %}
          {% set gas_cost = states('sensor.octopus_energy_gas__current_accumulative_cost') | float(default=0) %}
          {{ ((gas_cost * room_hours) / total) | round(2) if room_hours > 0 else 0 }}
        attributes:
          room_hours: "{{ states('sensor.master_bedroom_heating_time') }}"
          total_hours: "{{ states('sensor.total_heating_hours') }}"
          gas_cost: "{{ states('sensor.octopus_energy_gas__current_accumulative_cost') }}"
          percentage: "{{ ((states('sensor.master_bedroom_heating_time') | float(default=0) / states('sensor.total_heating_hours') | float(default=1)) * 100) | round(1) }}%"

      - name: landing_daily_cost
        unique_id: landing_daily_cost
        unit_of_measurement: "GBP"
        state: >
          {% set room_hours = states('sensor.landing_heating_time') | float(default=0) %}
          {% set total = states('sensor.total_heating_hours') | float(default=1) %}
          {% set gas_cost = states('sensor.octopus_energy_gas__current_accumulative_cost') | float(default=0) %}
          {{ ((gas_cost * room_hours) / total) | round(2) if room_hours > 0 else 0 }}
        attributes:
          room_hours: "{{ states('sensor.landing_heating_time') }}"
          total_hours: "{{ states('sensor.total_heating_hours') }}"
          gas_cost: "{{ states('sensor.octopus_energy_gas__current_accumulative_cost') }}"
          percentage: "{{ ((states('sensor.landing_heating_time
