# Obsidian Weather Templates

This repository contains two Templater-powered Obsidian notes that fetch weather data from
online APIs and render a simple forecast report.

Vibe-coded cus I have better things to do than master Javascript. 

## Files

- `weather-openmeteo.MD` - Uses the [Open-Meteo](https://open-meteo.com/) forecast API.
  - Geocodes a location name to coordinates.
  - Requests current weather, hourly humidity, daily weather codes, temperatures, sunrise/sunset.
  - Outputs a text report with emojis, temperature highs/lows, humidity, wind speed, and sun times.
  - Configurable location variable at the top of the file.

- `weather-wttr.MD` - Queries [wttr.in](https://wttr.in/) for weather data.
  - Parses JSON (`format=j1`) response.
  - Includes helpers for weekday names, moon phase emoji, condition emoji.
  - Displays current conditions with feels-like temperature, humidity, UV index.
  - Shows daily forecast lines with sunrise/sunset, UV, and moon phase.

## Usage

1. Install the Templater plugin in Obsidian.
2. Place these `.MD` files in a vault and open them.
3. Trigger Templater execution (e.g. via command pallet or hotkey) to populate the report.
4. Adjust the `location` variable inside each file to your desired city or place name.

### Notes

- The Open-Meteo template uses a small amount of inline JavaScript; modify it carefully.
- Open-Meteo does not provide moon phases; the WTTR template includes them instead.
- Sunrise/sunset is supported by the Open-Meteo API and displayed in 24‑hour format.

## Customization

Feel free to add more parameters to the API URLs or enhance the formatting.  These are
meant as starting points for quick weather notes.
