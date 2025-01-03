# Smart Heating Dashboard Sensors

Below is a curated set of **sensor** definitions that **directly power** the **Heating Control** dashboard you shared. These include:
* **History Stats** sensors (to track daily heating time per room)
* **Template** sensors (to calculate costs, total heating hours, and active heating rooms)

Also included is the **image** demonstrating the resulting dashboard UI.

---

## Dashboard Screenshot

![Smart Heating Dashboard Screenshot](https://preview.redd.it/smart-heating-just-got-smarter-v0-unt4d3987x0e1.jpeg?width=1080&crop=smart&auto=webp&s=9cd1ac8fb0dd8241d5829857b5c590a60e4f5b41)


---

# Smart Heating Dashboard: Step 1 – Requirements

Before setting up your **Smart Heating Dashboard**, you’ll need the following:

1. **Home Assistant**  
   - A running instance of Home Assistant (Core, Supervised, or OS).  
   - Basic knowledge of editing YAML configuration.

2. **Mushroom Cards**  
   - A popular custom Lovelace card collection that provides many nicely styled cards.  
   - [Mushroom GitHub Repo](https://github.com/piitaya/lovelace-mushroom) for installation instructions.

3. **Mini Graph Card**  
   - Used for rendering historical data graphs in a compact, elegant style.  
   - [Mini Graph Card Repo](https://github.com/kalkih/mini-graph-card) for installation steps.

4. **Climate Entities**  
   - One or more climate entities in Home Assistant, such as `climate.ethan`, `climate.jacob`, etc.  
   - These may come from smart thermostats, TRVs, or generic thermostat components.

5. **Gas Cost Sensor** (or Gas Consumption Sensor)  
   - Example: `sensor.octopus_energy_gas_e6s24887631961_4160990601_current_accumulative_cost`  
   - Your actual entity names may differ (e.g., from an integration like Octopus, MQTT, or manual input).

6. **HVAC Action Sensors**  
   - Template sensors that read `hvac_action` from each climate entity (e.g., `sensor.ethan_hvac_action`), enabling daily heating time tracking.

7. **History Stats**  
   - Utilized to track daily heating time for each room.  
   - Requires `history_stats` platform in your `sensor:` config.

8. **Environmental Sensors** (optional but recommended)  
   - For temperature and humidity, such as `sensor.upstairs_temperature_sensor_temperature` or `sensor.living_room_temperature_sensor_humidity`.  
   - Used to display environment graphs in the dashboard.

---

**Next Step**: Which part would you like next?  

You'll need the Code! It's in this repo:

Configuration YAML: SENSORS:  https://github.com/ITSpecialist111/Heating-Costs/blob/main/Sensors%20YAML

Dashboard YAML:  https://github.com/ITSpecialist111/Heating-Costs/blob/main/Dashbaord%20YAML  

Note:  Obviously change any sensors, names etc that you need too.
