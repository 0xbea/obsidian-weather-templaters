<%*
// initialize output buffer (templater provides tR but make sure it's defined)
if (typeof tR === "undefined") {
  tR = "";
}
var location = "new york"; // set to your location

// Geocode location to get coordinates
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

  var weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}` +
                   `&current_weather=true&hourly=relativehumidity_2m` +
                   `&daily=weathercode,temperature_2m_max,temperature_2m_min,sunrise,sunset` +
                   `&forecast_days=3&timezone=auto`;
  var weatherResponse = await requestUrl(weatherUrl);
  var weather = JSON.parse(weatherResponse.text);

  if (weather && weather.error) {
    tR += `Failed to retrieve weather data: ${weather.reason || JSON.stringify(weather)}`;
  } else {
    function getWeatherDescription(code) {
      var codeMap = {
        0: "Clear sky", 1: "Mainly clear", 2: "Partly cloudy", 3: "Overcast",
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

    function getDayOfWeek(dateStr) {
      var days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
      return days[new Date(dateStr).getDay()];
    }

    var current = weather.current_weather || {};
    var currentHumidity = "N/A";
    if (weather.hourly && weather.hourly.relativehumidity_2m) {
      currentHumidity = weather.hourly.relativehumidity_2m[0];
    }

    var currentCondition = getWeatherDescription(current.weathercode);
    var currentEmoji = weatherEmoji(current.weathercode);

    tR += `## 🌍 Weather Report for ${cityName}\n\n`;

    // Current conditions
    tR += `### Current Conditions\n`;
    tR += `${currentEmoji} **${currentCondition}** — ${current.temperature}°C, ${currentHumidity}% humidity, ${current.windspeed !== undefined ? current.windspeed + ' m/s' : 'N/A'} wind\n\n`;

    // 3-day forecast
    tR += `### 3-Day Forecast\n`;
    tR += `| Day | Date | Condition | High | Low | Sunrise-Sunset |\n`;
    tR += `| --- | ---- | --------- | ---- | --- | --------------- |\n`;
    
    var daily = weather.daily || {};
    var numDays = Math.min(3, (daily.time || []).length);
    for (var i = 0; i < numDays; i++) {
      var date = daily.time[i];
      var dow = getDayOfWeek(date);
      var condition = getWeatherDescription(daily.weathercode[i]);
      var condEmoji = weatherEmoji(daily.weathercode[i]);
      var high = daily.temperature_2m_max[i];
      var low = daily.temperature_2m_min[i];
      var sr = daily.sunrise && daily.sunrise[i] ? daily.sunrise[i].split("T")[1] : "N/A";
      var ss = daily.sunset && daily.sunset[i] ? daily.sunset[i].split("T")[1] : "N/A";
      tR += `| ${dow} | ${date} | ${condEmoji} ${condition} | ${high}°C | ${low}°C | ${sr} - ${ss} |\n`;
    }

    tR += `\n📅 Retrieved: ${new Date().toLocaleString()}\n`;
  }
}
%>