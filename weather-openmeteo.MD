<%*
// initialize output buffer (templater provides tR but make sure it's defined)
if (typeof tR === "undefined") {
  tR = "";
}
var location = "new york"; // set to your location

// Step 1: Geocode location to get coordinates
var geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(location)}&count=1&language=en&format=json`;
var geoResponse = await requestUrl(geoUrl);
var geoData = JSON.parse(geoResponse.text);
var geoResult = geoData && geoData.results && geoData.results[0];

if (!geoResult) {
  tR += `Location "${location}" not found`;
} else {
  var lat = geoResult.latitude;
  var lon = geoResult.longitude;
  var cityName = geoResult.name;
  if (geoResult.admin1) {
    cityName += ", " + geoResult.admin1;
  }

  // Step 2: Fetch weather data using coordinates
  // use valid parameters: current_weather and hourly humidity (and add sunrise/sunset)
  var weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}` +
                   `&current_weather=true&hourly=relativehumidity_2m` +
                   `&daily=weathercode,temperature_2m_max,temperature_2m_min,sunrise,sunset` +
                   `&timezone=auto`;
  var weatherResponse = await requestUrl(weatherUrl);
  var weather = JSON.parse(weatherResponse.text);

  if (weather && weather.error) {
    tR += `Failed to retrieve weather data: ${weather.reason || JSON.stringify(weather)}`;
    // stop further processing
  } else {

  // Helper: Convert WMO weather code to description
  function getWeatherDescription(code) {
    var codeMap = {
      0: "Clear sky",
      1: "Mainly clear", 2: "Partly cloudy", 3: "Overcast",
      45: "Foggy", 48: "Depositing rime fog",
      51: "Light drizzle", 53: "Moderate drizzle", 55: "Dense drizzle",
      61: "Slight rain", 63: "Moderate rain", 65: "Heavy rain",
      71: "Slight snow", 73: "Moderate snow", 75: "Heavy snow",
      77: "Snow grains",
      80: "Slight rain showers", 81: "Moderate rain showers", 82: "Violent rain showers",
      85: "Slight snow showers", 86: "Heavy snow showers",
      95: "Thunderstorm", 96: "Thunderstorm with slight hail", 99: "Thunderstorm with heavy hail"
    };
    return codeMap[code] || "Unknown";
  }

  // Helper: Get weekday name
  function getDayOfWeek(dateStr) {
    var days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
    return days[new Date(dateStr).getDay()];
  }

  // Helper: Weather condition emoji
  function weatherEmoji(code) {
    if (code === 0) return "🌞";
    if (code === 1 || code === 2) return "🌤️";
    if (code === 3) return "☁️";
    if (code === 45 || code === 48) return "🌫️";
    if (code >= 51 && code <= 67) return "🌧️";
    if (code >= 71 && code <= 86) return "🌨️";
    if (code >= 95) return "⛈️";
    return "🌈";
  }

  // extract current conditions (note property names from API)
  var current = weather.current_weather || {};
  var currentHumidity = "N/A";
  if (weather.hourly && weather.hourly.relativehumidity_2m) {
    // the first hourly value corresponds to current time (approx)
    currentHumidity = weather.hourly.relativehumidity_2m[0];
  }
  var currentCondition = getWeatherDescription(current.weathercode);
  var currentEmoji = weatherEmoji(current.weathercode);

  // Print header
  tR += `## 🌍 Weather Report for ${cityName}\n`;
  tR += `📅 Current Date & Time: ${new Date().toLocaleString()}\n`;

  // Current conditions summary
  tR += `Now: ${current.temperature}°C, ${currentEmoji} ${currentCondition}, Humidity: ${currentHumidity}%`;
  if (current.windspeed !== undefined) {
    tR += `, 🌬️ ${current.windspeed} m/s`;
  }
  tR += `\n\n`;

  // Loop through daily forecasts
  var daily = weather.daily || {};
  var i = 0;
  while (daily.time && i < daily.time.length) {
    var date = daily.time[i];
    var dow = getDayOfWeek(date);
    var condition = getWeatherDescription(daily.weathercode[i]);
    var condEmoji = weatherEmoji(daily.weathercode[i]);
    tR += `${dow} ${date} | ${condEmoji} ${condition} | 🌡️ H:${daily.temperature_2m_max[i]}°C L:${daily.temperature_2m_min[i]}°C`;
    if (daily.sunrise && daily.sunrise[i]) {
      // strip date portion, keep HH:MM
      var sr = daily.sunrise[i].split("T")[1];
      var ss = daily.sunset[i].split("T")[1];
      tR += ` | 🌄 ${sr} - 🌅 ${ss}`;
    }
    tR += `\n\n`;
    i++;
  }
  } // end else (weather not error)
}
%>
