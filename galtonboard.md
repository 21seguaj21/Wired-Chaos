# Galton Board

[← Back to index](index.md)

## Overview

This tutorial covers building a digital Galton Board using a Raspberry Pi 5 and a 64x32 RGB LED Matrix. You will simulate hundreds of particles bouncing through a peg interface to demonstrate how "chaos" naturally organizes into a bell curve.

You will learn:

- the historical significance of Sir Francis Galton’s discovery
- how to interface a Raspberry Pi 5 with high-density LED matrices
- how to write a physics simulation in Python using the `rgbmatrix` library
- how to troubleshoot specific Pi 5 hardware and power failures

<details>
<summary><strong>Jump to section</strong></summary>

- [History: The Law of Chaos](#history-the-law-of-chaos)
- [What you need](#what-you-need)
- [Step 1: The Raspberry Pi 5 Setup](#step-1-the-raspberry-pi-5-setup)
- [Step 2: Wiring the Matrix](#step-2-wiring-the-matrix)
- [Step 3: The Python Code](#step-3-the-python-code)
- [Common Problems: "It Just Stopped Working"](#common-problems-it-just-stopped-working)
- [Tips](#tips)

</details>

---

## History: The Law of Chaos

Invented by Sir Francis Galton in 1894, the Galton Board was designed to demonstrate that while individual events (a ball hitting a peg) are chaotic and unpredictable, the sum of many events is remarkably stable.

As balls drop through the rows of pegs, each hit forces a 50/50 choice: left or right. By the time the balls reach the bottom, the majority have performed an equal number of left and right bounces, landing in the center. The rare few that bounce only left or only right land at the edges. This creates the Normal Distribution—the famous "Bell Curve."

Galton famously said: *"Order in Apparent Chaos: I know of scarcely anything so apt to impress the imagination as the wonderful form of cosmic order expressed by the 'Law of Frequency of Error'."*

---

## What you need

- Raspberry Pi 5: The Pi 5 is powerful but requires specific timing fixes.
- 64x32 RGB LED Matrix: (P6 or P4 pitch works best).
- Adafruit RGB Matrix Bonnet: Highly recommended for clean wiring.
- 5V 4A Power Supply: The matrix pulls significant current; a standard Pi power supply isn't enough for both.
- MicroSD Card: Loaded with Raspberry Pi OS (64-bit).

---

## Step 1: The Raspberry Pi 5 Setup

The Pi 5 handles GPIO differently than previous models. Before coding, we must fix the timing and audio interference.

1.  Disable Audio: The Matrix uses the same hardware timers as the onboard audio.
    ```bash
    sudo nano /boot/firmware/config.txt
    ```
    Find `dtparam=audio=on` and change it to `off`.
2.  Add Pi 5 Fixes: Add this line to the bottom of the same file to prevent interrupt interference:
    ```text
    dtoverlay=gpio-no-irq
    ```
3.  Install the Library: Use the Adafruit automated installer:
    ```bash
    curl -sS [https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/rgb-matrix.sh](https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/rgb-matrix.sh) | bash
    ```
    *Select "Adafruit Bonnet" when prompted.*

---

## Step 2: Wiring the Matrix

1.  Plug your Adafruit Bonnet onto the Pi 5 GPIO pins.
2.  Connect the IDC Ribbon Cable from the Bonnet "Output" to the Matrix "Input."
3.  Connect the 4-pin Power Cable to the Matrix and screw the other end into the Bonnet's power terminals.
4.  Plug your 5V 4A Power Supply into the DC Jack on the Bonnet. This will power both the Matrix and the Pi via the GPIO.

---

## Step 3: The Python Code

Create a file named `galton.py`. The `gpio_slowdown` setting is the most important line for Pi 5 users.

```python
import time
import random
from rgbmatrix import RGBMatrix, RGBMatrixOptions

# 1. Configuration
options = RGBMatrixOptions()
options.rows = 32
options.cols = 64
options.chain_length = 1
options.parallel = 1
options.hardware_mapping = 'adafruit-hat'
options.gpio_slowdown = 4  # ESSENTIAL for Raspberry Pi 5
options.drop_privileges = False

matrix = RGBMatrix(options = options)
canvas = matrix.CreateFrameCanvas()

# 2. Game State
bins = [0] * 64
pegs = []

# Generate peg positions (a triangle)
for y in range(4, 20, 2):
    for x in range(32 - y, 32 + y, 4):
        pegs.append((x, y))

def run_simulation():
    global canvas
    while True:
        ball_x, ball_y = 32, 0
        
        while ball_y < 31:
            canvas.Clear()
            
            # Draw Pegs
            for px, py in pegs:
                canvas.SetPixel(px, py, 255, 255, 255) # White pegs
            
            # Draw Accumulated Bins
            for x, height in enumerate(bins):
                for h in range(height):
                    canvas.SetPixel(x, 31 - h, 0, 0, 255) # Blue stacked balls
            
            # Logic: Falling and Bouncing
            if (ball_x, ball_y + 1) in pegs:
                ball_x += 1 if random.random() > 0.5 else -1
            
            ball_y += 1
            
            # Stop if we hit the floor or a stack
            if ball_y >= (31 - bins[ball_x]):
                bins[ball_x] += 1
                break
                
            # Draw Falling Ball
            canvas.SetPixel(ball_x, ball_y, 255, 0, 0) # Red ball
            canvas = matrix.SwapOnVSync(canvas)
            time.sleep(0.01)

            # Reset if full
            if max(bins) > 12:
                bins[:] = [0] * 64

try:
    run_simulation()
except KeyboardInterrupt:
    print("Stopping...")

## Common Problems: "It Just Stopped Working"

Hardware projects on the Pi 5 are temperamental. If your board suddenly goes dark or displays "static," check these common failure points:

### The "Flicker of Death"
* The Problem: The image appears, but it flickers wildly or has horizontal lines.
* The Fix: The Pi 5 is too fast. Increase `options.gpio_slowdown` to `5` or `6`. Also, ensure you disabled the audio in `config.txt`.

### Sudden Blackout / Script Freezing
* The Problem: The simulation was running, and then it just stopped or the screen turned black.
* The Cause: Power Surge. As the "bins" fill up, more LEDs turn on. This increases the Amp draw. If your power supply is weak, the voltage drops, and the Pi or the Matrix controller crashes.
* The Fix: Use a dedicated 5V 4A power supply. Do NOT power the matrix through the Pi’s USB port.

### Permission Denied
* The Problem: You get an error saying you can't access memory.
* The Fix: The matrix library requires root access. You must run the script with `sudo`:
    ```bash
    sudo python3 galton.py
    ```

### The "Zombie" Pixels
* The Problem: Random pixels stay lit even after you clear the screen.
* The Fix: This is usually a loose IDC (ribbon) cable. The Pi 5’s vibrations from the fan can sometimes wiggle these loose. Unplug and reseat the cable firmly.

---

## Tips

* Heat Management: The Pi 5 runs hot when processing matrix math. Use the official Active Cooler fan to prevent the Pi from slowing down mid-simulation.
* Speed: If the simulation is too slow, decrease the `time.sleep(0.01)` value in the main loop.
* Color: Modify the code to make the falling ball change color based on whether it bounces left or right!

[← Back to index](index.md)
