--
layout: page
title: "Sample page"
permalink: /sample/
--

Most applications include code details to interact with external systems.  Testing these interactions, especially in non-production systems, can be unpredictable.  Feature flags provide a way to swap the real integration details with code that fakes different states of an external system.  This avoids interruptions when those systems are offline, not responding as expected, or just hard to change.

Let's imagine a `Thermostat` application that, among other things, determines if the outside temperature is between two values:

```kotlin
class Thermostat(private val temperatureService: TemperatureService) {
    fun isBetween(minTemp: Int, maxTemp: Int): Boolean {
        val currentTemp = temperatureService.get()
        return currentTemp > minTemp && currentTemp < maxTemp
    }
}

interface TemperatureService {
    fun get(): Int
}
```

and so on
