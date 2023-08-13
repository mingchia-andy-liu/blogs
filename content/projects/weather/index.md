---
title: "Another Weather App"
date: 2020-08-14
draft: false
Summary: "A weather app with air quality built by me for me"
slug: "weather-report-project"
---

{{< lead >}}
Yet another weather report app
{{< /lead >}}
Another weather report

[{{< icon "link" >}} https://weather.aliu.dev/](https://weather.aliu.dev/)

This report uses your ip location from Cloudflare to find the weather report + air quality. This project was created in a time where the city I lived in had severe forest fire. The air quality was terrible for the whole summer. When you step outside, you can feel the ash when you breath in. I wanted to see the air quality of the day to see if I want to go outside.

The weather data is from [openweather](https://openweathermap.org/). They have pretty comprehensive data including air quality which is what I need. After that, I added search for any address location. This was actually quite interesting problem space: reverse geocoding. I ended up using [Nominatim](https://nominatim.org/). 

Features
- Weather report 
- Air quality report
- Look up by addresses 

The UI is simple, just location and the weather information. Perfectly fits my needs to see it on mobile devices too.

### Screenshot

{{<figure src="screenshot.png" alt="A screenshot of the alert(1)" >}}
