# Galton Board

[← Back to index](index.md)

## Overview

This tutorial covers building a digital **Galton Board** using a **Raspberry Pi 5** and a **64×32 RGB LED Matrix**. The project simulates hundreds of particles moving through rows of pegs to demonstrate how many random events can form a predictable bell-shaped distribution [10][17].

You will learn:

- the historical significance of **Sir Francis Galton** and the Galton Board [10][39]
- how a Galton Board demonstrates probability and the normal distribution [10][17]
- how the **Central Limit Theorem** relates to repeated random events [22]
- how to interface a **Raspberry Pi 5** with an LED matrix [24][26]
- how to write the simulation in **Python** or **C++** [12][36]
- how to troubleshoot common hardware, software, and power problems

---

## History: Sir Francis Galton and the Normal Distribution

The **Galton Board**, also known as a **Galton box** or **quincunx**, is associated with British scientist **Sir Francis Galton**. The device demonstrates how repeated random events can create an organized statistical pattern [10][39].

A traditional Galton Board consists of rows of pegs. Balls are released from the top and bounce either left or right as they collide with each row of pegs. After many balls have fallen through the board, they collect into bins along the bottom [10].

Although the path of any individual ball is unpredictable, the overall collection of balls tends to form a **bell-shaped distribution**.

### How the Probability Works

At each row of pegs, the ball effectively makes one of two possible movements:

- **left**
- **right**

If both directions are equally likely, each movement can be modeled with a probability of approximately:

\[
P(\text{left}) = 0.5
\]

\[
P(\text{right}) = 0.5
\]

The final location of a ball depends on the total combination of left and right movements it makes while falling through the board.

Balls that make a similar number of left and right movements tend to land near the center. More extreme sequences, such as repeatedly moving in one direction, are less common and therefore place fewer balls near the edges.

After many trials, the resulting shape resembles the **Normal Distribution**, commonly called the **Bell Curve** [17].

The mathematics behind repeated random events approaching a predictable distribution is related to the **Central Limit Theorem** [22].

Pearson's work on the mathematical treatment of error and observations also contributed to the historical development of statistical distributions [21].

For additional background, see:

- **Galton Board visualization:** [10]
- **Normal distribution and standard deviation:** [17]
- **Central Limit Theorem:** [22]
- **Sir Francis Galton:** [39]

---

## What You Need

- **Raspberry Pi 5** — the computer that runs the simulation and controls the display [24][26]
- **64×32 RGB LED Matrix** — displays the pegs, falling particles, and accumulated distribution
- **RGB Matrix adapter or Bonnet** — provides a cleaner connection between the Raspberry Pi GPIO pins and the matrix
- **5V high-current power supply** — provides sufficient power for the LED matrix
- **MicroSD Card** — stores Raspberry Pi OS and the project files
- **IDC ribbon cable** — connects the matrix controller to the LED panel

The Raspberry Pi 5 provides considerably more processing capability than earlier Raspberry Pi models and can be used for applications that require real-time graphical calculations [24].

Official Raspberry Pi documentation should be consulted for current Raspberry Pi hardware and software configuration information [26].

---

## Critical Warnings and What to Avoid

> **Important:** Always verify the voltage, current requirements, and wiring specifications of your exact LED matrix and controller before applying power.

- **DO NOT power a high-current LED matrix directly from ordinary GPIO pins.**
- **NEVER change matrix power wiring while the system is powered on.**
- **DO NOT reverse the positive and ground connections.**
- Double-check the required voltage before connecting the matrix.
- Make sure the power supply can provide enough current for the LED matrix.
- Shut down the Raspberry Pi before changing GPIO or matrix connections.
- Use Raspberry Pi 5-compatible software and drivers where possible.

The Raspberry Pi documentation provides additional information about GPIO, power, operating systems, and Raspberry Pi hardware [26].

---

## Step 1: Raspberry Pi 5 Setup

Install a current **64-bit Raspberry Pi OS** on the MicroSD card.

Raspberry Pi maintains official documentation covering installation, configuration, GPIO, and operating system setup [26].

After starting the Raspberry Pi, update the operating system:

```bash
sudo apt update
sudo apt upgrade -y
```
If your RGB matrix library requires additional Raspberry Pi 5 configuration, follow the installation instructions supplied by the library or matrix-controller manufacturer.

Because Raspberry Pi hardware and software continue to change, current documentation should be checked before modifying low-level GPIO or boot configuration settings [26].

## Step 2: Wiring the Matrix

A typical setup follows these general steps:

1. Shut down and disconnect power from the Raspberry Pi.
2. Attach the RGB Matrix adapter or Bonnet to the Raspberry Pi GPIO header.
3. Connect the **IDC ribbon cable** from the controller output to the LED matrix input.
4. Connect the matrix power cable.
5. Double-check the positive and ground connections.
6. Connect the appropriate external power supply.
7. Power the system on only after all connections have been checked.

### Basic Connection Layout

```text
Raspberry Pi 5
      │
      ▼
RGB Matrix Adapter / Bonnet
      │
      ├──────── IDC Ribbon Cable ───────► RGB LED Matrix
      │
      └──────── Power Connection ───────► RGB LED Matrix
```
Consult Raspberry Pi documentation for GPIO and board information [26].



## Step 3: Python Simulation

Python is a general-purpose programming language commonly used for scripting, education, scientific computing, and hardware projects [36].

Python also supports reusable libraries that provide additional functionality without requiring programmers to write everything from scratch [37].

Create a file named:galton.py

Example:

```python
import random
import time
from rgbmatrix import RGBMatrix, RGBMatrixOptions

# --- Matrix Configuration ---
options = RGBMatrixOptions()
options.rows = 32
options.cols = 64
options.chain_length = 1
options.parallel = 1
options.hardware_mapping = 'regular'  # Change to 'adafruit-hat' if using Adafruit bonnet
options.gpio_slowdown = 4             # Necessary for Pi 5 speed timing

matrix = RGBMatrix(options=options)
canvas = matrix.CreateFrameCanvas()

WIDTH = 64
HEIGHT = 32
BIN_MAX_HEIGHT = 10  # Bottom 10 rows reserved for stacked balls
PEG_START_Y = 4
PEG_END_Y = 20

# Generate triangular peg layout
PEGS = set()
for y in range(PEG_START_Y, PEG_END_Y, 2):
    # Center section expands outwards as y goes deeper
    offset = (y - PEG_START_Y) // 2
    center_x = WIDTH // 2
    for x in range(center_x - offset * 2, center_x + offset * 2 + 1, 4):
        if 0 <= x < WIDTH:
            PEGS.add((x, y))

# Bins state (stores ball stack count per column)
bins = [0] * WIDTH

def run_simulation():
    while True:
        # Start ball near top center
        ball_x = float(WIDTH // 2)
        ball_y = 1.0
        
        # Color palette for active falling ball (Cyan/Yellow glow)
        ball_color = (255, 200, 0)

        # Fall animation
        while ball_y < (HEIGHT - BIN_MAX_HEIGHT):
            # Check for bounce when aligned with a peg row
            if (int(ball_x), int(ball_y)) in PEGS or (int(ball_x) + 1, int(ball_y)) in PEGS:
                # 50/50 probability bounce left or right
                ball_x += -1.0 if random.random() < 0.5 else 1.0

            ball_y += 0.5
            ball_x = max(0.0, min(float(WIDTH - 1), ball_x))

            # Draw current state
            draw_frame(ball_x, ball_y, ball_color)
            time.sleep(0.015)

        # Drop into bottom bin column
        bin_col = int(round(ball_x))
        bin_col = max(0, min(WIDTH - 1, bin_col))

        if bins[bin_col] < BIN_MAX_HEIGHT:
            bins[bin_col] += 1

        # Reset histogram if any bin fills up completely
        if max(bins) >= BIN_MAX_HEIGHT:
            time.sleep(1.5)
            bins[:] = [0] * WIDTH

def draw_frame(ball_x, ball_y, ball_color):
    canvas.Clear()

    # 1. Draw Pegs (Dim Blue/White)
    for px, py in PEGS:
        canvas.SetPixel(px, py, 60, 60, 100)

    # 2. Draw Stacked Bins (Gradient based on height)
    for col in range(WIDTH):
        stack_height = bins[col]
        for h in range(stack_height):
            y_pos = (HEIGHT - 1) - h
            # Color gradient: green base transitioning to red peaks
            red = min(255, h * 25)
            green = max(0, 255 - h * 20)
            canvas.SetPixel(col, y_pos, red, green, 50)

    # 3. Draw Active Falling Ball
    bx, by = int(round(ball_x)), int(round(ball_y))
    if 0 <= bx < WIDTH and 0 <= by < HEIGHT:
        canvas.SetPixel(bx, by, *ball_color)

    # Swap double-buffered frame canvas
    global canvas
    canvas = matrix.SwapOnVSync(canvas)

if __name__ == '__main__':
    try:
        print("Starting 64x32 Galton Board... Press Ctrl+C to stop.")
        run_simulation()
    except KeyboardInterrupt:
        matrix.Clear()
        print("\nSimulation exited.")
```

# How the Galton Board Simulation Works

The Python script recreates a physical Galton Board (also known as a bean machine) [10] on a 64x32 RGB LED matrix by combining physics simulation, probability generation, and real-time matrix rendering.

Below is a breakdown of how the program translates mathematical theory into a visual display:

### 1. Matrix Initialization & Configuration
The script establishes a hardware interface with the 64x32 HUB75 RGB matrix using double-buffered frame canvases [26]. Double buffering prevents display flickering by drawing the next frame in memory before swapping it onto the physical display in sync with the refresh rate.

### 2. Triangular Peg Lattice Generation
A virtual triangular lattice of pegs is generated programmatically in the upper two-thirds of the grid (rows 4 to 20) [10]. The pegs spread outward as the row index deepens, mirroring Pascal’s triangle and providing the obstacle field required to produce a normal distribution [17].

### 3. Central Ball Injection
Each particle (representing a dropped marble) is spawned at the top center of the matrix [10] with float-precision coordinates. This ensures that every ball starts with an equal probability of leaning left or right upon its first collision.

### 4. Stochastic Bounce Engine
As the ball moves downward, collision checks evaluate whether its position intersects peg coordinates [10]. When a collision occurs, a pseudo-random number generator decides whether the ball bounces left ($-1.0$ x-offset) or right ($+1.0$ x-offset) with an exact $50\%$ probability ($p = 0.5$) [22].

### 5. Accumulation & Histogram Mapping
When a ball clears the bottom peg row, its final horizontal coordinate determines which of the 64 vertical bins it falls into [10]. The script increments the bin’s counter, stacking pixels from the bottom edge upward to visually form a bell curve (Gaussian distribution) over repeated drops [17, 21, 22].

### 6. Dynamic Color Rendering & Auto-Reset
Each frame renders three distinct visual layers in real time [10]: dim blue pixels for static pegs, a bright yellow pixel for the active falling ball, and a dynamic green-to-red gradient for the histogram bins. When any bin reaches maximum capacity, the array resets automatically to begin a new collection cycle.

---
## Step 4: C++ Simulation

The same Galton Board simulation can also be implemented using **C++**. 

C++ is a compiled programming language widely used when programmers need direct hardware access, low-level memory management, and precise execution performance [12].

### Source Code (`galton.cpp`)

Create a file named `galton.cpp` and insert the following code:

```cpp
#include "led-matrix.h"

#include <unistd.h>
#include <cstdlib>
#include <ctime>

using rgb_matrix::RGBMatrix;
using rgb_matrix::Canvas;

int main(int argc, char *argv[]) {
    srand(time(nullptr));

    // Matrix configuration
    RGBMatrix::Options options;
    options.rows = 32;
    options.cols = 64;
    options.hardware_mapping = "adafruit-hat";

    RGBMatrix *matrix = RGBMatrix::CreateFromOptions(options, argc, argv);
    if (matrix == nullptr) {
        return 1;
    }

    Canvas *canvas = matrix->CreateFrameCanvas();
    int bins[64] = {0};

    // Main simulation loop
    while (true) {
        int bx = 32;
        int by = 0;

        while (by < 31) {
            canvas->Clear();

            // 1. Draw static pegs
            for (int y = 4; y < 20; y += 2) {
                for (int x = 32 - y; x < 32 + y; x += 4) {
                    canvas->SetPixel(x, y, 255, 255, 255);
                }
            }

            // 2. Draw accumulated particles in histogram bins
            for (int x = 0; x < 64; x++) {
                for (int h = 0; h < bins[x]; h++) {
                    canvas->SetPixel(x, 31 - h, 0, 0, 255);
                }
            }

            // 3. Calculate stochastic bounce logic
            if (by >= 4 && by < 20 && by % 2 == 0) {
                if (rand() % 2 == 0) {
                    bx--;
                } else {
                    bx++;
                }
            }

            // Keep particle within boundary constraints
            if (bx < 0)  bx = 0;
            if (bx > 63) bx = 63;

            by++;

            // 4. Check for landing / stack collision
            if (by >= 31 - bins[bx]) {
                bins[bx]++;
                break;
            }

            // 5. Render falling active particle
            canvas->SetPixel(bx, by, 255, 0, 0);

            // Frame refresh & animation timing
            canvas = matrix->SwapOnVSync(canvas);
            usleep(10000); // 10ms delay
        }

        // Auto-reset when any bin exceeds maximum height threshold
        if (bins[bx] > 12) {
            sleep(1);
            for (int i = 0; i < 64; i++) {
                bins[i] = 0;
            }
        }
    }

    delete matrix;
    return 0;
}
```
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
```
---

## Tips

* **Heat Management:** The Pi 5 runs hot when processing matrix math. Use the official **Active Cooler** fan to prevent the Pi from slowing down mid-simulation.
* **Speed:** If the simulation is too slow, decrease the `time.sleep(0.01)` (Python) or `usleep(10000)` (C++) value.
* **Color:** Modify the code to make the falling ball change color based on whether it bounces left or right!

[← Back to index](index.md)
