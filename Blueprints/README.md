# Charger State Tracker Blueprint

A Home Assistant blueprint that tracks the state of battery chargers based on power consumption.

## Features

- **Automatic State Tracking**: Monitors power consumption and transitions between Idle, Charging, and Complete states
- **Configurable Thresholds**: Adjust power threshold and timing delays to match your chargers
- **Optional Notifications**: Get notified when charging completes
- **Debouncing**: Prevents false state changes from power fluctuations

## Prerequisites

Before using this blueprint, you need to create an **Input Select Helper**:

### Option 1: Via UI (Recommended)
1. Go to **Settings** → **Devices & Services** → **Helpers**
2. Click **Create Helper** → **Dropdown**
3. Configure:
   - **Name**: `Chargers Status` (or your preferred name)
   - **Options**: Add these three options in order:
     - `Idle`
     - `Charging`
     - `Complete`
   - **Icon**: `mdi:battery-charging` (optional)
4. Click **Create**

### Option 2: Via configuration.yaml
Add this to your `configuration.yaml`:

```yaml
input_select:
  chargers_status:
    name: Chargers Status
    options:
      - Idle
      - Charging
      - Complete
    initial: Idle
    icon: mdi:battery-charging
```

Then restart Home Assistant.

## Installation

1. **Import the Blueprint**:
   - Go to **Settings** → **Automations & Scenes** → **Blueprints**
   - Click **Import Blueprint**
   - Paste the URL to this blueprint file OR
   - Copy the blueprint YAML content and manually create it

2. **Create an Automation from the Blueprint**:
   - Go to **Settings** → **Automations & Scenes** → **Automations**
   - Click **Create Automation** → **Use Blueprint**
   - Select **Charger State Tracker**

## Configuration

### Required Settings

- **Power Consumption Sensor**: Select your power monitoring sensor
  - Example: `sensor.chargers_smart_plug_electric_consumption_w`
  
- **Status Helper**: Select the input_select helper you created
  - Example: `input_select.chargers_status`

### Optional Settings (with defaults)

- **Power Threshold**: `4W` - Power level that indicates charging is active
- **Charging Start Delay**: `5 seconds` - Debounce time before marking as "Charging"
- **Charging Complete Delay**: `30 seconds` - How long power must be low before marking as "Complete"
- **Idle Reset Delay**: `300 seconds (5 minutes)` - How long to stay in "Complete" before resetting to "Idle"

### Notification Settings (Optional)

- **Enable Notifications**: Toggle to enable
- **Notification Service**: e.g., `notify.mobile_app_your_phone`
- **Notification Title**: Customize the notification title
- **Notification Message**: Customize the notification message

## Usage

### In Dashboards

Add the status to your dashboard:

```yaml
type: entity
entity: input_select.chargers_status
name: Charger Status
```

Or use a conditional card to show different icons:

```yaml
type: conditional
conditions:
  - entity: input_select.chargers_status
    state: Charging
card:
  type: markdown
  content: "🔋 Charging in progress..."
```

### In Automations

Trigger actions based on the charger state:

```yaml
trigger:
  - platform: state
    entity_id: input_select.chargers_status
    to: 'Complete'
action:
  - service: light.turn_on
    target:
      entity_id: light.notification_light
    data:
      rgb_color: [0, 255, 0]
```

### In Scripts

Check the current state:

```yaml
condition:
  - condition: state
    entity_id: input_select.chargers_status
    state: 'Charging'
```

## State Transitions

```
┌──────┐   Power > Threshold    ┌──────────┐   Power < Threshold   ┌──────────┐
│ Idle │ ────────────────────> │ Charging │ ───────────────────> │ Complete │
└──────┘                        └──────────┘                       └──────────┘
   ↑                                                                      │
   └──────────────────────────────────────────────────────────────────────┘
                        After Idle Reset Delay
```

## Troubleshooting

### States not changing
- Check that your power sensor is reporting values correctly
- Verify the power threshold is appropriate for your chargers
- Increase the debounce delays if you have fluctuating power readings

### False "Complete" triggers
- Increase the "Charging Complete Delay"
- Some chargers have trickle charge phases that may drop below threshold

### Notifications not working
- Verify the notification service name is correct
- Test the notification service independently first
- Check Home Assistant logs for errors

## Example Configuration

For a Zooz ZEN04 monitoring battery chargers:

```
Power Consumption Sensor: sensor.chargers_smart_plug_electric_consumption_w
Status Helper: input_select.chargers_status
Power Threshold: 4W
Charging Start Delay: 5 seconds
Charging Complete Delay: 30 seconds
Idle Reset Delay: 300 seconds
Enable Notifications: Yes
Notification Service: notify.mobile_app_iphone
```

## Advanced Usage

### Create a Template Sensor for the Power State

Add to `configuration.yaml` for additional visibility:

```yaml
template:
  - sensor:
      - name: "Chargers Power State"
        unique_id: chargers_power_state
        state: >
          {% set power = states('sensor.chargers_smart_plug_electric_consumption_w') | float(0) %}
          {% if power >= 4 %}
            charging
          {% else %}
            idle
          {% endif %}
```

### Track Charging Duration

Create a timer helper that starts when charging begins:

```yaml
timer:
  charger_duration:
    name: Charging Duration
    duration: "12:00:00"
```

Then add automations to start/stop it based on the charger state.

## Support

If you encounter issues or have suggestions:
1. Check the Home Assistant logs for errors
2. Verify all prerequisites are met
3. Test with default settings first before customizing

## License

Free to use and modify for personal use.
