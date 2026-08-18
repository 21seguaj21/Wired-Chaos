# Galton Board

[← Back to index](index.md)

## Overview

This tutorial covers building a digital Galton Board using a Raspberry Pi 5 and a 64x32 RGB LED Matrix. You will simulate hundreds of particles bouncing through a peg interface to demonstrate how "chaos" naturally organizes into a bell curve.

You will learn:

- the historical significance of Sir Francis Galton’s discovery
- how to interface a Raspberry Pi 5 with high-density LED matrices
- how to write a physics simulation in **Python** or **C++**
- how to troubleshoot specific Pi 5 hardware and power failures

<details>
<summary><strong>Jump to section</strong></summary>

- [History: The Law of Chaos](#history-the-law-of-chaos)
- [What you need](#what-you-need)
- [Step 1: The Raspberry Pi 5 Setup](#step-1-the-raspberry-pi-5-setup)
- [Step 2: Wiring the Matrix](#step-2-wiring-the-matrix)
- [Step 3: The Python Code](#step-3-the-python-code)
- [Step 4: The C++ Code](#step-4-the-c-code)
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

- [Raspberry Pi 5](https://www.adafruit.com/product/6007): The Pi 5 is powerful but requires specific timing fixes.
- [64x32 RGB LED Matrix](https://www.amazon.com/gp/aw/d/B0B2ZC85KN/?_encoding=UTF8&pd_rd_plhdr=t&aaxitk=5e1d9c6708e98905b70869acaccf2e05&hsa_cr_id=0&qid=1787091302&sr=1-1-9e67e56a-6f64-441f-a281-df67fc737124&i=aps&aref=48hUlqwPPA&ref_=sbx_s_sparkle_sbtcd_asin_0_title&pd_rd_w=VjAFl&content-id=amzn1.sym.8de9b3d5-f5c5-40e9-9b39-d65f08d6ea68%3Aamzn1.sym.8de9b3d5-f5c5-40e9-9b39-d65f08d6ea68&pf_rd_p=8de9b3d5-f5c5-40e9-9b39-d65f08d6ea68&pf_rd_r=73P6YMCDHSCVH252K3YX&pd_rd_wg=WejXD&pd_rd_r=7b9cba90-b1b1-472f-883d-b11f65fde344): (P6 or P4 pitch works best).
- [Adafruit RGB Matrix Bonnet](https://www.adafruit.com/product/3211?srsltid=AfmBOorS4FCz2Mq10TKf70Fh0zKMQnmzZtXNlrWiERu_ovk4de6TeQ6o): Highly recommended for clean wiring.
- [5V 4A Power Supply](https://www.amazon.com/Arkare-100-240V-Replacement-Security-Raspberry/dp/B0F24141Z1/ref=sr_1_2_sspa?dib=eyJ2IjoiMSJ9.xyxhP44Op2PtzScDyUA9of9ECKSACOhaleRAlhVldJQijUGrRuJ_nsuHirIqMRHH9rm9uAWbfNv78PLUMHvSh71tp_yTt5VRyxk8H9TGVI506FfiZMeoEaPRJZjtrFeve5mBA2UupYYRwLIEg9wrZg5-_Okg5ocoYf9xM8P5r9VPnpmrZOwp0FfgM_NQsP67P0ISEJSTmLrY_pB-S8udYjz7lqQ0TgWa9t2Iqxj-1TqDSDaqgD4hwaw69Jo5Ag9CEYzKpcdp5xagfRGnSHzlAeM5kMAfR_qK2vSynhOT7SA.2q7QGRpGqUxOZJ-piLJEILkQSoK0r-OIrUa7jb_Mki4&dib_tag=se&hvadid=695274587949&hvdev=c&hvexpln=67&hvlocphy=9013387&hvnetw=g&hvocijid=13606180190065508269--&hvqmt=e&hvrand=13606180190065508269&hvtargid=kwd-295316582550&hydadcr=18888_13357672&keywords=5v+4a+power+supply&mcid=7dd64de846fb370c819667a75d028a8f&qid=1787091373&s=electronics&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1): The matrix pulls significant current; a standard Pi power supply isn't enough for both.
- [MicroSD Card](https://www.adafruit.com/product/5249): Loaded with Raspberry Pi OS (64-bit).

---

## Critical Warnings & What to Avoid

- DO NOT power the LED matrix through the Raspberry Pi's GPIO pins: A 64x32 panel drawing full brightness can pull up to 4A, which can permanently damage the Pi or cause constant system crashes.
- NEVER plug or unplug power cables or the Bonnet while powered on: Hot-plugging high-current DC power can cause voltage spikes that destroy the sensitive shift registers on the LED matrix.
- DO NOT connect power in reverse polarity: Double-check your power supply wiring at the Bonnet's screw terminals (+5V to Red, GND to Black), as swapped wires will instantly fry the hardware.
- DO NOT power the Pi via USB-C and the Bonnet simultaneously: Feed power strictly through the 5V screw terminals on the Adafruit Bonnet to avoid competing power rails.
- DO NOT use outdated matrix drivers: Legacy libraries built for Pi 3 or Pi 4 do not recognize the Pi 5's new RP1 I/O controller and will fail to run.
- AVOID running on-board audio drivers: Raspberry Pi sound drivers interfere with the precise hardware PWM microsecond timing required to display a flicker-free image.

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
## Step 4: The C++ Code

For higher performance and smoother physics, you can use C++. Save this file as `galton.cpp`.

```cpp
#include "led-matrix.h"
#include <unistd.h>
#include <vector>
#include <stdlib.h>

using rgb_matrix::RGBMatrix;
using rgb_matrix::Canvas;

int main(int argc, char *argv[]) {
    RGBMatrix::Options defaults;
    defaults.rows = 32;
    defaults.cols = 64;
    defaults.hardware_mapping = "adafruit-hat";
    defaults.gpio_slowdown = 4; // Essential for Pi 5

    RGBMatrix *matrix = RGBMatrix::CreateFromOptions(defaults, argc, argv);
    Canvas *canvas = matrix->CreateFrameCanvas();
    int bins[64] = {0};

    while (true) {
        int bx = 32, by = 0;
        while (by < 31 - bins[bx]) {
            canvas->Clear();
            
            // Draw Pegs
            for (int y = 4; y < 20; y += 2) 
                for (int x = 32-y; x < 32+y; x+=4) canvas->SetPixel(x, y, 255, 255, 255);
            
            // Draw Bins
            for (int x = 0; x < 64; x++)
                for (int h = 0; h < bins[x]; h++) canvas->SetPixel(x, 31-h, 0, 0, 255);

            // Bouncing logic
            if (by >= 4 && by < 20 && by % 2 == 0) bx += (rand() % 2 == 0) ? 1 : -1;
            by++;

            canvas->SetPixel(bx, by, 255, 0, 0);
            canvas = matrix->SwapOnVSync(canvas);
            usleep(10000);
        }
        bins[bx]++;
        if (bins[bx] > 12) for(int i=0; i<64; i++) bins[i] = 0;
    }
    return 0;
}
**To compile and run:**

```bash
g++ galton.cpp -lrgbmatrix -lpthread -o galton
sudo ./galton
## Common Problems: "It Just Stopped Working"

Hardware projects on the Pi 5 are temperamental. If your board suddenly goes dark or displays "static," check these common failure points:

### The "Flicker of Death"
* **The Problem:** The image appears, but it flickers wildly or has horizontal lines.
* **The Fix:** The Pi 5 is too fast. Increase `options.gpio_slowdown` to `5` or `6`. Also, ensure you disabled the audio in `config.txt`.

### Sudden Blackout / Script Freezing
* **The Problem:** The simulation was running, and then it just stopped or the screen turned black.
* **The Cause:** **Power Surge.** As the "bins" fill up, more LEDs turn on. This increases the Amp draw. If your power supply is weak, the voltage drops, and the Pi or the Matrix controller crashes.
* **The Fix:** Use a dedicated **5V 4A** power supply. Do **NOT** power the matrix through the Pi’s USB port.

### Permission Denied
* **The Problem:** You get an error saying you can't access memory.
* **The Fix:** The matrix library requires root access. You must run the script with `sudo`:

```bash
sudo python3 galton.py
### The "Zombie" Pixels
* **The Problem:** Random pixels stay lit even after you clear the screen.
* **The Fix:** This is usually a loose **IDC (ribbon) cable**. The Pi 5’s vibrations from the fan can sometimes wiggle these loose. Unplug and reseat the cable firmly.

---

## Tips

* **Heat Management:** The Pi 5 runs hot when processing matrix math. Use the official **Active Cooler** fan to prevent the Pi from slowing down mid-simulation.
* **Speed:** If the simulation is too slow, decrease the `time.sleep(0.01)` (Python) or `usleep(10000)` (C++) value.
* **Color:** Modify the code to make the falling ball change color based on whether it bounces left or right!

[← Back to index](index.md)
