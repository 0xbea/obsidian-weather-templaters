<%*
const location = "new york";
const response = await requestUrl(`https://wttr.in/${location}$?format=j1`);
const data = JSON.parse(response.text);
console.log(data);

// Helper: Get weekday name
function getDayOfWeek(dateStr) {
  const days = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
  return days[new Date(dateStr).getDay()];
}

// Helper: Moon phase emoji
function moonEmoji(phase) {
  const map = {
    "New Moon": "🌑",
    "Waxing Crescent": "🌒",
    "First Quarter": "🌓",
    "Waxing Gibbous": "🌔",
    "Full Moon": "🌕",
    "Waning Gibbous": "🌖",
    "Last Quarter": "🌗",
    "Waning Crescent": "🌘"
  };
  return map[phase] || "🌙";
}

// Helper: Weather condition emoji
function weatherEmoji(condition) {
  const c = condition.toLowerCase();
  if (c.includes("sunny") || c.includes("clear")) return "🌞";
  if (c.includes("cloud") || c.includes("overcast")) return "🌥️";
  if (c.includes("rain") || c.includes("drizzle")) return "🌧️";
  if (c.includes("thunder")) return "⛈️";
  if (c.includes("snow")) return "🌨️";
  if (c.includes("fog") || c.includes("mist") || c.includes("haze")) return "🌫️";
  if (c.includes("storm")) return "🌩️";
  return "🌈"; // default/fallback emoji
}

// Extract current observation date/time from wttr.in
const current = data.current_condition?.[0] || {};
const obsDateTime = current.localObsDateTime || "Unknown time";
const currentCondition = current.weatherDesc?.[0]?.value || "N/A";
const currentEmoji = weatherEmoji(currentCondition);
const capitalized = location.charAt(0).toUpperCase() + location.slice(1);

// Print current local date & time
tR += `## 🌍 Weather Report for ${capitalized}\n`;
tR += `📅 Current Date & Time: ${obsDateTime}\n`;

// Current conditions summary
tR += `Now: ${current.temp_C}°C (Feels like ${current.FeelsLikeC}°C), ${currentEmoji} ${currentCondition}, Humidity: ${current.humidity}%, ☀️ UV: ${current.uvIndex}\n\n`;

// Loop through daily forecasts
for (let day of data.weather) {
  const dow = getDayOfWeek(day.date);
  const astro = day.astronomy?.[0] || {};
  const moon = `${moonEmoji(astro.moon_phase)} ${astro.moon_phase} (${astro.moon_illumination}%)`;

  // Using midday condition (~12:00-15:00)
  const condition = day.hourly?.[4]?.weatherDesc?.[0]?.value || "N/A";
  const condEmoji = weatherEmoji(condition);

  tR += `${dow} ${day.date} | ${condEmoji} ${condition} | 🌡️ H:${day.maxtempC}°C L:${day.mintempC}°C | 🌄 ${astro.sunrise}-${astro.sunset} (${day.sunHour}h) 🌅 | UV: ${day.uvIndex} | ${moon} \n\n`;
}
%>