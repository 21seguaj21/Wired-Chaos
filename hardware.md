# Hardware

[← Back to index](index.md)

---

## Arduinos 

Arduino is an open-source electronics platform that consists of hardware and software designed to make it easy for anyone to create interactive projects. The core of the platform is the Arduino microcontroller board, which can read inputs from sensors and control outputs like motors, LEDs, and other devices. Arduino boards are programmable using the Arduino IDE, which uses a simplified version of C/C++ programming language. They are widely used in prototyping, education, and hobbyist projects due to their affordability, simplicity, and extensive community support.

Key points for beginners:
- Arduino boards come in various models like Uno, Nano, and Mega, each with different sizes, pin counts, and features.
- They typically operate at 5V or 3.3V and can be powered via USB or an external power supply.
- Programming is done through a USB connection using the Arduino IDE, which is free and available for multiple operating systems.
- Unlike computers, Arduinos don't run an operating system; they execute simple programs in a continuous loop.
- They have digital and analog pins for connecting sensors, buttons, LEDs, and other components.
- Extensive libraries and add-on boards (shields) are available to extend functionality.
- The Arduino community provides abundant tutorials, forums, and example projects for learning.

## Raspberry Pi 

Raspberry Pi is a series of small, affordable single-board computers developed by the Raspberry Pi Foundation. It's about the size of a credit card and runs a full operating system like Raspberry Pi OS (based on Linux). It features GPIO (General Purpose Input/Output) pins that allow it to interface with sensors, motors, LEDs, and other electronic components, making it ideal for electronics projects, robotics, and IoT applications. Raspberry Pis are widely used in education for teaching programming and computer science, as well as in DIY projects, media centers, and even industrial applications. Popular models include the Raspberry Pi 4 and 5, with varying amounts of RAM, processing power, and connectivity options like Wi-Fi and Bluetooth.

Key points for beginners:
- Raspberry Pis run a full Linux-based operating system, allowing them to function like a regular computer.
- They can connect to a monitor, keyboard, and mouse to serve as a desktop PC for browsing, coding, and media playback.
- Storage is provided via a microSD card, which holds the OS and files.
- Power is supplied through a micro USB port, typically requiring a 5V adapter.
- Different models (like Pi 4, Pi 5) offer varying CPU speeds, RAM, and features for different needs.
- GPIO pins enable connection to electronic components for projects, similar to Arduino.
- It's great for learning programming languages like Python, and has a large community with tutorials and projects.
- Can be used headless (without a screen) for servers, IoT devices, or remote access via SSH.

## Stacker Game Wiring

For the stacker game, wire the Arduino so the LEDs, button, and any buzzer share the same power and ground rails.

- Arduino 5V -> breadboard +5V rail
- Arduino GND -> breadboard GND rail
- LEDs:
  - LED 1 anode -> digital pin `2`
  - LED 2 anode -> digital pin `3`
  - LED 3 anode -> digital pin `4`
  - LED 4 anode -> digital pin `5`
  - LED 5 anode -> digital pin `6`
  - Each LED cathode -> resistor -> GND
  - Use 220 Ω resistors for each LED
- Button:
  - One side -> digital pin `7`
  - Other side -> GND
  - Use a 10 kΩ pull-up resistor to 5V, or enable `INPUT_PULLUP` in code
- Buzzer / speaker (optional):
  - Positive -> digital pin `8`
  - Negative -> GND

Important:
- All components must share the same ground.
- If you use `INPUT_PULLUP`, wire the button to GND and use the internal pull-up instead of an external resistor.

## Snake Game Wiring

The snake game can use LEDs for the game board and one or more buttons for direction or start.

- Arduino 5V -> breadboard +5V rail
- Arduino GND -> breadboard GND rail
- LEDs:
  - Use 6–8 LEDs (or more if your design supports it)
  - Example pin mapping:
    - Head LED -> digital pin `2`
    - Body LED 1 -> digital pin `3`
    - Body LED 2 -> digital pin `4`
    - Body LED 3 -> digital pin `5`
    - Body LED 4 -> digital pin `6`
    - Tail LED -> digital pin `7`
  - Each LED cathode -> 220 Ω resistor -> GND
- Buttons:
  - Move button A -> digital pin `8`
  - Move button B -> digital pin `9`
  - Start/reset button -> digital pin `10`
  - Use pull-down or `INPUT_PULLUP` wiring
  - If using `INPUT_PULLUP`, connect button to GND and read low when pressed
- Optional buzzer:
  - Positive -> digital pin `11`
  - Negative -> GND

Notes:
- If using multiple direction buttons, make sure each button has a stable default state.
- Keep the wiring neat so the LED array is easy to follow.

## Galton Board

This section is not done yet.

- Placeholder for Galton board wiring
- Add details later for:
  - motor/pin connections
  - sensor inputs
  - power and ground routing
  - physical layout