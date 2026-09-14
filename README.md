# 🐠 Aquarium Control Blueprint

**Version 1.1**

A smart, reliable Home Assistant blueprint for automated aquarium lighting with sunrise/sunset simulation and synchronized CO2 injection.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FDiaoul%2Fhass-aquarium-control%2Fblob%2Fmain%2Faquarium_control.yaml)

## ✨ Features

- 🌅 **Sunrise/Sunset Simulation** - Gradual brightness transitions mimic natural lighting cycles
- ☀️ **Configurable Brightness** - Set maximum brightness and minimum starting brightness
- 📅 **Daily Schedule** - Automatically repeats every day at your configured start time
- ⏱️ **Flexible Timing** - Configure transition durations and max brightness duration
- 🔧 **Maintenance Mode** - Toggle to full brightness for feeding/cleaning, CO2 automatically turns off for safety
- 🫧 **CO2 Synchronization** - CO2 injection turns on/off synchronized with lighting schedule
- ⏰ **CO2 Time Offset** - Configurable offset (default: 60 min before lights) for optimal CO2 buildup before photosynthesis
- 🌊 **Smooth Transitions** - Gentle brightness changes for stress-free lighting adjustments
- 🔄 **Smart Updates** - 60-second incremental brightness updates for reliable Zigbee light control
- 💾 **State Recovery** - Automatically recovers correct state after Home Assistant restarts
- 🎛️ **Multiple Lights** - Support for controlling multiple synchronized light entities
- 🔌 **Availability Handling** - Automatically detects and recovers when lights or CO2 switch come back online
- 🛟 **Fault Tolerant** - One unresponsive light never blocks the rest of the tank

## 📦 Installation

Click the button above to import the blueprint directly into your Home Assistant.

**Requires Home Assistant 2024.10 or newer.**

## 🔧 Required Helper Entities

Before creating the automation, you need to create three input helper entities. These allow you to control the aquarium settings from your dashboard.

Go to **Settings** → **Devices & Services** → [**Helpers**](https://my.home-assistant.io/redirect/helpers/)

### 1. Input Datetime - Daily Start Time

Create a **Date and/or time** helper:
- **Name:** Aquarium Start Time (or your preference)
- **Has date:** ❌ Disabled
- **Has time:** ✅ Enabled
- **Example entity ID:** `input_datetime.aquarium_start_time`

### 2. Input Number - Total Light Duration

Create a **Number** helper:
- **Name:** Aquarium Light Duration (or your preference)
- **Minimum:** 4
- **Maximum:** 12
- **Step size:** 0.5
- **Unit of measurement:** hours
- **Example entity ID:** `input_number.aquarium_light_duration`

### 3. Input Number - Max Brightness

Create a **Number** helper:
- **Name:** Aquarium Max Brightness (or your preference)
- **Minimum:** 1
- **Maximum:** 100
- **Step size:** 1
- **Unit of measurement:** %
- **Example entity ID:** `input_number.aquarium_max_brightness`

💡 **Tip:** Set initial values (e.g., start time: 08:00, duration: 9 hours, brightness: 60%) before creating the automation.

## 🚀 Quick Start

1. **Create the three required helper entities** (see above section)
2. **Create an automation** from the blueprint
3. **Select your helper entities:**
   - Daily start time entity (input_datetime you created)
   - Total light duration entity (input_number in hours)
   - Max brightness entity (input_number in %)
4. **Configure your aquarium light entities** (required)
5. **Configure durations:**
   - Transition duration (sunrise/sunset simulation, default: 30 minutes)
6. **(Optional)** Adjust min brightness (default: 1)
7. **(Optional)** Configure maintenance mode toggle (input_boolean, switch, or binary_sensor)
8. **(Optional)** Configure CO2 switch and time offset

The blueprint organizes all settings into collapsible sections for easy configuration.

## 🧠 How It Works

The blueprint creates a natural lighting cycle for your aquarium:

1. **CO2 turns on** at configured offset (default: 60 min before light start)
2. **Lights turn on** to minimum brightness, then **fade in** gradually to maximum brightness over transition duration
3. **Lights stay at max** brightness (total duration minus fade-in and fade-out)
4. **Lights fade out** gradually from maximum to minimum brightness over transition duration, then **turn off**
5. **CO2 turns off** at same offset before light ends
6. **Repeats every day** at the configured start time

**Transition Behavior:**
- At cycle start: Lights instantly turn on to minimum brightness, then smoothly fade to maximum
- During cycle: Smooth brightness changes every minute with gentle 5-second transitions
- At cycle end: Lights smoothly fade to minimum brightness, then transition off
- This creates natural sunrise/sunset simulation while ensuring lights are completely off when not in use

If your total duration is shorter than two transitions, the transition is clamped to half the total duration so the cycle never runs longer than you configured.

**Maintenance Mode:**
- When toggled on, overrides schedule and sets lights to full brightness
- CO2 automatically turns off for safety during maintenance
- Resume normal schedule by toggling maintenance mode off

**CO2 Timing:**
- CO2 offset is negative (e.g., -60 minutes means CO2 starts 60 min BEFORE lights)
- Allows CO2 to build up in the water before photosynthesis begins
- CO2 also turns off early (same offset before lights end)
- Recommended: -60 minutes for planted tanks

## 🎛️ Configuration Tips

- **Start Time (Dashboard):** Adjust your input_datetime helper to change when lights start. Perfect for seasonal adjustments!
- **Total Light Duration (Dashboard):** Set your input_number between 4-12 hours. Typical planted tanks: 8-10 hours. Easily experiment to find your sweet spot!
- **Max Brightness (Dashboard):** Control with your input_number (1-100%). Start at 60-80% and adjust based on plant growth and algae. Lower = less algae.
- **Transition Duration (Blueprint):** 30-45 min works well for most tanks. Longer transitions (60 min) create ultra-gradual sunrise/sunset.
- **Light Transition (Blueprint):** 5 seconds default provides gentle brightness changes. Set to 0 if your lights don't support transitions.
- **Min Brightness (Blueprint):** Brightness level for sunrise start and sunset end (1-20%). Lights fade between this minimum and your maximum brightness. Keep at 1-5% for dramatic sunrise/sunset effect.
- **CO2 Offset (Blueprint):** -60 minutes default works well for most planted tanks.

## 📊 Example Daily Cycle

With example settings from your dashboard helpers:
- Start time: 08:00 (from input_datetime)
- Total duration: 9 hours (from input_number)
- Max brightness: 60% (from input_number)
- Transition: 30 min (from blueprint)
- CO2 offset: -60 min (from blueprint)

**Daily Timeline:**
- **07:00** - CO2 turns on
- **08:00** - Lights turn on to minimum brightness and begin fading in (sunrise)
- **08:30** - Lights reach max brightness (60%)
- **16:00** - CO2 turns off
- **16:30** - Lights begin fading out (sunset)
- **17:00** - Lights turn off (cycle complete)

💡 **Change anytime from your dashboard!** Adjust start time, duration, or brightness without editing the automation.

## 🔧 Maintenance Mode Helper (Optional)

For convenience, create an input_boolean helper for maintenance mode:

**1. Create Helper**

Go to **Settings** → **Devices & Services** → [**Helpers**](https://my.home-assistant.io/redirect/helpers/)

Create a **Toggle** helper:
- Name: "Aquarium Maintenance Mode"
- Icon: mdi:wrench

**2. Configure in Blueprint**

Select your input_boolean in the **Maintenance Mode Toggle** field.

**Usage:** Toggle ON for feeding/cleaning (full brightness, CO2 off), toggle OFF to resume schedule.

## ⚙️ Technical Details

- **Update Frequency:** Every 60 seconds for smooth, reliable transitions
- **Recovery:** Automatically recovers correct state after HA restart or device reconnection
- **Zigbee Reliability:** 60-second updates prevent command flooding that can cause Zigbee issues
- **Phase Calculation:** Recalculates current phase and brightness on every update for accuracy
- **Availability Handling:** Skips unavailable devices and automatically syncs when they come back online
- **Fault Tolerance:** A light that fails to respond does not block the remaining lights or the CO2 switch in the same update
- **Helper Fallback:** If the start time helper is unavailable, the schedule falls back to 08:00 instead of failing every update

## 🤝 Support

If you encounter issues:
- Review automation traces in **Settings** → **Automations & Scenes** → _your automation_ → **Traces**
- Test templates in **Developer Tools** → **Template**
- Open an issue on [GitHub](https://github.com/Diaoul/hass-aquarium-control/issues)

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

Made with ❤️ for the Home Assistant community
