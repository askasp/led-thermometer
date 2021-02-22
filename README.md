# LED thermometer for Africa burn

![Image of sauna and thermometer](https://i.imgur.com/A6VTrcJ.png)
For Africa Burn 2019 my camp (Vagabonds) decided to gift a sauna to the community. This is a quick and dirty walkthrough for making the LED thermometer as seen on the picture. The hotter it gets the more LEDs are lit, and redder they become. In the picture the temperature is 60°C and the range is 0-80°C


## Equipment

| What             | Which                                                                                                                                                                                       | Link |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------|
| Power Supply     | PHEVOS 5v 12A Dc Universal Switching Power Supply for Raspberry PI Models,CCTV                                                                                                              | tbd  |
| Micro Controller | Rpi 3                                                                                                                                                                                       | tbd  |
| Thermometer      | Vktech 2M Waterproof Digital Temperature Temp Sensor Probe DS18b20                                                                                                                          | tbd  |
| Copper Wire      | UL1015 Commercial Copper Wire, Bright, Red, 22 AWG, 0.0253" Diameter, 100' Length (Pack of 1) (As the leds draws about 6AMP its nice/wise to have a bit more then the standard RPI jumpers) | tbd  |


## Wiring

![Wiring](https://i.imgur.com/NnVQD9A.png)
## Code
The code is rather strighforward as it builds upon NeoPixel and w1Thermsensor. 
Only small modifications were needed from the NeoPixel examples. Note the MAX and MIN constants as they decide the temperature range for the termometer

```
#!/usr/bin/env python3

import time
from rpi_ws281x import *
import argparse
import math
from w1thermsensor import W1ThermSensor
sensor = W1ThermSensor()

MAX_TEMP = 100
MIN_TEMP = 0


# LED strip configuration:
LED_COUNT      = 300      # Number of LED pixels.
MAX_COLOR_LED = 200
LED_PIN        = 18      # GPIO pin connected to the pixels (18 uses PWM!).
#LED_PIN        = 10      # GPIO pin connected to the pixels (10 uses SPI /dev/spidev0.0).
LED_FREQ_HZ    = 800000  # LED signal frequency in hertz (usually 800khz)
LED_DMA        = 10      # DMA channel to use for generating signal (try 10)
LED_BRIGHTNESS = 255     # Set to 0 for darkest and 255 for brightest
LED_INVERT     = False   # True to invert the signal (when using NPN transistor level shift)
LED_CHANNEL    = 0       # set to '1' for GPIOs 13, 19, 41, 45 or 53

def getColor(lednr):
red = math.floor(lednr/MAX_COLOR_LED * 255)
print("red is",red)
print("lednr is",lednr)
if red > 255:
    red = 255

blue = 255-red

if blue > red:
    green = 255 - blue
else:
    green = 255 -red
    red = 255
    blue = 0

return Color(red,green,blue)

def SetColor(temp):
nr_of_leds =math.floor(LED_COUNT*temp/MAX_TEMP)
print("nr of leds is",nr_of_leds)
for i in range(0,LED_COUNT):
    if i < nr_of_leds:
        color = getColor(i)
    else:
        color = Color(0,0,0)
    strip.setPixelColor(i,color)
    strip.show()


# Main program logic follows:
if __name__ == '__main__':
# Process arguments
parser = argparse.ArgumentParser()
parser.add_argument('-c', '--clear', action='store_true', help='clear the display on exit')
args = parser.parse_args()

# Create NeoPixel object with appropriate configuration.
strip = Adafruit_NeoPixel(LED_COUNT, LED_PIN, LED_FREQ_HZ, LED_DMA, LED_INVERT, LED_BRIGHTNESS, LED_CHANNEL)
# Intialize the library (must be called once before other functions).
strip.begin()

print ('Press Ctrl-C to quit.')
if not args.clear:
    print('Use "-c" argument to clear LEDs on exit')

try:
    while True:
        SetColor(math.floor(sensor.get_temperature()))
        time.sleep(15)
      

except KeyboardInterrupt:
    if args.clear:
        colorWipe(strip, Color(0,0,0), 10)

```

