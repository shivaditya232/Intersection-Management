# Smart Intersection Management

An adaptive traffic signal controller for a four-way intersection, built on an ESP32 with an I2C LCD. Instead of running fixed-length phases, it extends the green time for whichever road has more vehicles waiting, and handles pedestrian crossing requests.

## How it works

**Vehicle counting.** Each road has a push button that stands in for a vehicle sensor. Counts are only accepted while that road is **red**, so the controller knows how much traffic has built up before the light turns.

**Adaptive green time.** Base green is 10 seconds, extended by how many vehicles are queued:

| Vehicles waiting | Green time |
| --- | --- |
| 0–4 | 10 s |
| 5–9 | 20 s |
| 10–14 | 30 s |
| 15+ | 40 s |

**Pedestrian requests.** Pressing the pedestrian button schedules a dedicated pedestrian green phase (8 s, all vehicle signals red) at the next phase change, after which the opposite road gets its green.

**Phases** cycle through NS green → NS yellow → EW green → EW yellow, with a pedestrian phase inserted on request. Yellow is a fixed 3 seconds.

## Display

The LCD shows the live state of the intersection:

```
NSG 10+20s
T=30  EW=14
```

Line 1 is the active green phase and its base + extra time; line 2 is the countdown and the number of vehicles queued on the other road.

## Hardware

| Component | Detail |
| --- | --- |
| Board | ESP32 DevKit C v4 |
| Display | 16×2 LCD over I2C (`0x27`) — SDA on GPIO 32, SCL on GPIO 33 |
| North–South LEDs | Red 2, Yellow 4, Green 5 |
| East–West LEDs | Red 18, Yellow 19, Green 21 |
| Pedestrian LEDs | Red 22, Green 23 |
| Buttons | NS count 12, EW count 13, Pedestrian request 14 |

## Simulating it

The project is set up for [Wokwi](https://wokwi.com), so no physical hardware is needed — `diagram.json` describes the full circuit and `main.cpp` is the sketch. Requires the **LiquidCrystal I2C** library.

## Note on implementation

Timing is handled by a `waitOneSecondWithButtons()` helper that polls the buttons while it waits, rather than `millis()`. This keeps the phase logic readable and means button presses are never missed during a countdown.
