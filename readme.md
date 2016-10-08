# esp_lightingnado

> I watched Sharknado 4 yesterday so, for some reason, lightingnado came to mind for a project name.

## Dependencies

* ArduinoJson (64)
* AsyncMqttClient (346)
* ESPAsyncTCP (305)
* NeoPixelBus (547)
* WifiManager (567)

## Pins

I'm using an ESP12E DEVKIT V2 (ESP + USB programmer board) so my pins correspond to that:

* **RESET (D7)** - Momentarily grounding this pin within the first couple seconds of startup resets the configuration and enables the captive configuration portal
* **LED DATA (RXA)** - This pin is used for the data signal for the WS2812 LED strip.
* **POWER SUPPLY OUTPUT (D6)** - set to high when power supply should be on
* **POWER SUPPLY STATUS (D5)** - should be high if power supply is on

## Controlling it

The device will connect to the configured MQTT server and operate on two topics.

* **esp/(node name)/status** publishes states messages when it connects to MQTT and disconnects, and when the lighting mode changes
* **esp/(node name)/control** accepts control messages to change lighting modes (see below for samples)

## Lighting Modes

The lighting mode is changed by pushing messages onto the control channel.  The messages always start with a single character (because that scales...) that represents the target lighting mode and is followed by additional data (usually JSON) to pass into that lighting mode.

If the specific lighting mode is already active it'll perform an update on the lighting mode rather than switching to it.  This allows for animated transitions between states within a lighting mode.

### Ants

Marching ants.  Sorta..  Takes an array of colors and then lays those out in a repeating pattern on the strip and then cycles them left or right.

```
A{
  "delay": 100,
  "colors": [
    [100,0,0],
    [0,100,0],
    [0,0,100]
  ],
  right: true
}
```

* **Delay** is the delay (ms) between updates
* **Right** are we moving right? or left?
* **Colors** is an array of RGB sets for the colors on the strip


### Percent

This one turns the lighting strip into a big progress bar.  Changes are animated (both color and percent).

```
P{"percent":10, "color":[255,0,0], delay: 1}
```

* **Percent** is the percentage (0..100) of the light string to illuminate
* **Color** is the target color for the lit lights
* **Delay** is the delay (ms) between updates (when changing from one state to another it moves 1% / update)

### Slide

This one isn't very exciting... It lights up a few lights in a particular color and then slides them left or right.

```
S{"delay": 100, "length": 5, "right": false, "color":[200,100,0]}
```

* **Delay** is the delay (ms) between updates
* **Length** is the number of lights to illuminate
* **Right** are we moving right? or left?
* **Color** color for the lit lights
* **Step** is the number of pixels to jump each update

### Twinkle

This one is a completely photo realistic 1 dimensional star field.  Pixels light up randomly and then decay back to black.  Oooh!

```
T{"delay": 10, "rate": 0.1, "decay": 5}
```

* **Delay** is the delay (ms) between updates
* **Rate** is the probability (0..1) that a star will be spawned in any given update
* **Decay** is the rate (1..255) the star intensity drops per update


### Rainbow

This one yields a pretty cycling rainbow

```
R{"delay": 10, "right": true, "step": 5, "brightness": 75}
```

* **Delay** is the delay (ms) between updates
* **Right** is it sliding right... or left
* **Brightness** is brightness from 0 to 100 (%)
* **Step** is the number of pixels to jump each iteration

### Tracker

This lights up specified pixels and fades them out

```
K{"data": [1,2,3,100], "decay": 10, "color": [255,0,0]}
```

* **Data** LEDs to turn on (0-100 - scales for LED string)
* **Color** [R,G,B] for color
* **Decay** Rate at which the colors drop off
