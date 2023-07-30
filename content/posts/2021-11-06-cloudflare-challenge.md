---
title: "Cloudflare Developer Challenge: Air Quality Report & Prize"
date: 2021-11-06T11:45:42-07:00
images: images/weather-2.jpeg
slug: cloudflare-dev-challenge
tags:
  - cloudflare
type: post
---

## Introduction

Back in the summer, Cloudflare launches a developer challenge where if anyone build their applications use *two* services from Cloudflare's developer platform, they could win a prize. I built a localized weather report that can detect your location and tell you the weather and the air quality. In this post I'll talk a little about the challenge and my entry.


## What to build

Looking over their developer platform they have so many new cool toys to play with: workers, KV, wranglers, etc. Even more challenging is what I should build with all these tools. I immediately know that I want to use worker in some way. It is so versatile that it has so many potentials.

I started readying the API docs and noticed that you can know the **latitude** and **longitude** of the request. This was really cool because usually you have to do some IP reverse decoding to find out the location. I thought to myself I need to use these properties.

When I started to build the exploring API, the real world was not great. When I lived, we were having a lot of 🔥 heat waves and the air quality was not great. All the forest fire had caused the ashes to be blown into the city. And I thought I could build an app that tells me the air quality of your current location! This would've been something I could use.


## Air Quality Report

With the goal in mind, I needed a data source that is free and easy to use. I found out [OpenWeather](https://openweathermap.org/) has an air quality API. Both Cloudflare and OpenWeather's API were really easy to use and build.

Here is my first iteration, an extremely simple air quality report. (I censored the city and country). The air quality is color coded according to the [national standard](https://webcam.srs.fs.fed.us/test/AQI.shtml). I was very happy to see what I have done.

{{< figure
    src="/images/weather-1.png"
    alt="Air quality first iteration"
>}}


## Improvements

### Geocoding

I came back to the project the next week. I wanted to do more. I was thinking of searching the air quality based on any location. The problem is how can I translate any arbitrary location string to a fixed location with lat/lon. Turns out this is called [geocoding](https://en.wikipedia.org/wiki/Address_geocoding).

> Geocoding, is the process of taking a text-based description of a location, such as an address or the name of a place, and returning geographic coordinates, frequently latitude/longitude pair, to identify a location on the Earth's surface.

This is something new to me. I started to see if there was any free API that can help me to do this. Lo and behold, I found [Nominatim](https://nominatim.org/), the open-source geocoding with OpenStreetMap Data. This was amazing. It is free without an api key requirement. The results were usually really good. It can also do non-English descriptions, Awesome!

### Weather

Now that I have geocoding, I felt like I should also include the weather report. I was already using the OpenWeather API. I might as well. The problem is the air quality is on its own API endpoint `/air_pollution` instead of the all in one endpoint `/onecall`. Instead of making 2 API calls which would double my usage. I tried to see if I can calculate the air quality myself.

Finding out the air pollution formula was **HARD**. It seems like there is no 1 given formula. I ended up only using the pm25 metric from [airnowtech](https://forum.airnowtech.org/t/the-aqi-equation/169).

### Emoji

Just for fun, I changed the emoji next to the title to reflect the weather report and time of the day.

```js
  // clear sky
  "01": { n: "🌚", d: "😎" },
  // 	few clouds
  "02": { n: "🌑", d: "🌤" },
  // scattered clouds
  "03": { n: "☁️", d: "☁️" },
  // 	broken clouds
  "04": { n: "⛅️", d: "⛅️" },
  // 	shower rain
  "09": { n: "🌧", d: "🌧" },
  // rain
  "10": { n: "🌦", d: "🌦" },
  // thunderstorm
  "11": { n: "⛈", d: "⛈" },
  // 	snow
  "13": { n: "❄️", d: "❄️" },
  // 	mist
  "50": { n: "🌫", d: "🌫" },
```

## 🏁 Entry
Here is what I submitted. Feel free to play with it as well. It has a weather report, air quality index, uv index, and location based search.
* Source Code – https://github.com/mingchia-andy-liu/weather-report
* Live Demo – https://weather.aliu.dev/


## 🏆 Result

After the submission, I totally forgot about it. I received an email titled "`[Cloudflare]: Summer Challenge Winner!`". I was so surprised after 20 days. I didn't even think my little application was worth anything. I was so happy. I followed up with the email and entered my shipping address.

The prize came after another 10 days.

{{< figure
    src="/images/weather-2.jpeg"
    alt="Cloudflare prize in a box."
>}}

I am really happy that I challenged myself to do this. Even though it is a small application, I learnt a little more about geocoding and air quality index than I did before.
