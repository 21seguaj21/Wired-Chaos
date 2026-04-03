# Snake

[← Back to index](index.md)

## Overview

This tutorial shows how to build a Snake game on an 8x8 LED matrix using the `LedControl` library and an analog joystick.

You will learn:

- how to wire the MAX7219 matrix and joystick
- how the Arduino sketch moves the snake without blocking
- how food, collision detection, and restart logic work
- common problems and how to fix them

<details>
<summary><strong>Jump to section</strong></summary>

- [What does `void` mean?](#what-does-void-mean)
- [What is a library?](#what-is-a-library)
- [What you need](#what-you-need)
- [Step 1: Install the library](#step-1-install-the-library)
- [Step 2: Wire the hardware](#step-2-wire-the-hardware)
- [Step 3: Understand the sketch structure](#step-3-understand-the-sketch-structure)
- [Step 4: Pin and game variables](#step-4-pin-and-game-variables)
- [Step 5: Initialize the display and inputs](#step-5-initialize-the-display-and-inputs)
- [Step 6: Read joystick input and choose a direction](#step-6-read-joystick-input-and-choose-a-direction)
- [Step 7: Move the snake without blocking](#step-7-move-the-snake-without-blocking)
- [Step 8: Update the snake body](#step-8-update-the-snake-body)
- [Step 9: Handle collisions](#step-9-handle-collisions)
- [Step 10: Spawn food safely](#step-10-spawn-food-safely)
- [Step 11: Show game over and restart](#step-11-show-game-over-and-restart)
- [Full sketch reference](#full-sketch-reference)
- [Line-by-line explanation](#line-by-line-explanation)
- [Common problems and fixes](#common-problems-and-fixes)

</details>

## What does `void` mean?

In Arduino C++, `void` is a function return type that means "nothing." When a function is declared as `void`, it does not return any value back to the caller.

In this sketch, `void setup()` and `void loop()` are special Arduino functions that the board calls automatically:

- `void setup()` runs once at startup and does not return anything.
- `void loop()` runs repeatedly and also does not return a value.

Using `void` is normal for functions that perform actions like initializing hardware or updating the game state without producing a result.

## What is a library?

In Arduino programming, a library is a collection of pre-written code that provides functions and tools to simplify common tasks. For example, the `LedControl` library handles communication with the MAX7219 LED driver chip, allowing you to control the matrix without writing low-level SPI code from scratch.

Libraries are installed via the Arduino IDE's Library Manager and included in your sketch with `#include <LibraryName.h>`. They extend Arduino's capabilities, such as controlling displays, sensors, or motors, making projects like this Snake game easier to build.

Explanation:
- Libraries save time by reusing code for hardware interactions.
- Without libraries, you'd need to implement protocols like SPI manually, which is error-prone and complex.
- In this tutorial, `LedControl` abstracts the matrix control, so you can focus on game logic.

Try this:
- Open the Arduino IDE and browse the Library Manager to see popular libraries like `Servo` or `Wire`.
- As a challenge, find the `LedControl` source code online and identify the `setLed()` function to understand how it works internally.

## What you need

- Arduino board (Uno, Nano, etc.) - [Buy On Adafruit](https://www.adafruit.com/product/2488)
- 9V battery clip with 5.5mm/2.1mm plug - [Buy On Adafruit](https://www.adafruit.com/product/80?gad_source=1&gad_campaignid=23438252138&gbraid=0AAAAADx9JvSzl8Zwe_PvvlEqgxzDMWtat&gclid=Cj0KCQjwyr3OBhD0ARIsALlo-OlHya_VpHYPBQZNaTq0mcxw_bNJ4wg3DxYBWZ-KisDtH-hRyNIa5qQaApWfEALw_wcB)
- 9V Battery - [Buy On Adafruit](https://www.google.com/aclk?sa=L&ai=DChsSEwi-1s2fptKTAxW_wp8JHRC7KYkYACICCAEQBhoCbWQ&co=1&ase=2&gclid=Cj0KCQjwyr3OBhD0ARIsALlo-OkZ5bcJ2kaCJucQ7CM-WFjatWCXsCKQF5kVA9DIs0tspzK4S_BgYu4aAoEzEALw_wcB&cid=CAASugHkaHZYuCchP4cNnNJ5WnR2QJBqcVmrQKg9tWa1grMBDa6SJmBY2A8jk1PlX__p3rVaXNlEbtwlEIESDmbCca-07XV9WP13-do2hn9VwFUHJM2hmEplgmJNlNGe9nZEJxf56PlCsB0p2UobYDEQiKuqDD_5C6vzf8k69tH7ViptjU095fkiaHFXBvDXOBp7fcySnCXQphcdLEPCOrM0vU2KPXXnzTXLNsA-rAl_mtIy-NNTMXoa9NY54ls&cce=2&category=acrcp_v1_32&sig=AOD64_2ut9HYXtbhVhvZ3VmjfJIuazjS2A&ctype=5&q=&nis=4&ved=2ahUKEwja88efptKTAxXprYkEHe4oI-IQ5bgDKAB6BAgIEAs&adurl=)
- 8x8 LED matrix with MAX7219 driver - [Buy On Amazon](https://www.amazon.com/sspa/click?ie=UTF8&spc=MTozNTQzMDcxNDE1MTQ3MTA2OjE3NzUyNDA0MjE6c3BfYXRmOjIwMDA5MTE4ODU4MTcxMTo6MDo6&url=%2FHiLetgo-MAX7219-Matrix-Display-Control%2Fdp%2FB07W6KZR5D%2Fref%3Dsr_1_1_sspa%3Fcrid%3D2WA7SUKVVS82N%26dib%3DeyJ2IjoiMSJ9.ZyHxd8WbxVSm4mZYyszaNNNuhcoATQLIWdgXpkkFx2H8lUMvbg0SnpUf9U77BbpRvwmPY2Oooi8-TCaB5--aFIopTz0GN3Dw6ciZI2r5Dh790ZXCEbi4kggajJFhmfQMarZLEzHlH3gJUKD_SYBVBM4h3otrhYvgoCECdoVrXxP0I0TrhmrwNcSkVdTrRB9UNtTdLBgcp4msjAYzWhPxoHoNhX7QYt-KBmB9RXFVJrltlOzmqbC1hSJbyLw_D0haxWeN3ZY1WuXN2Lw-66nHAm3hbzlq1FIIIkHOiGtBTvk.hPjjj95xfUkdNns58r2X1JGYcy3SH-uM4u3KKIfmK8Y%26dib_tag%3Dse%26keywords%3D8x8%2Bled%2Bmatrix%26qid%3D1775240421%26sprefix%3D8x8%2Bled%2Bm%252Caps%252C162%26sr%3D8-1-spons%26sp_csd%3Dd2lkZ2V0TmFtZT1zcF9hdGY%26psc%3D1)
- Joystick module with X/Y analog outputs and push button - [Buy On Amazon](https://www.amazon.com/Joystick-Module-Arduino-ESP8266-Raspberry/dp/B0DQ37P5RQ/ref=sr_1_2?crid=3MRPR7RJHPRJN&dib=eyJ2IjoiMSJ9.bBQi4scrgxtb6hdiI0MLF5ydm51WAc-kZwQFy6_Te2aYUeZW4tDJ8oEfPg2aI2HnQxC5oeVWLSosvWd7XihH5BVccA-pRfY4ABh_LM2mxtk10r_yEqbYJiiP0cXR2-TyzUw31wmZlr9bfkSuM3xO25TjLpGqVA_vaNjXNM-e1fzi50w_h2Lozn4v8Hv9eL2nK8OxxYFde3idUvf5Q-5S_LvOffnRw-fMOqd9M3MIeH4.DnF0kNMn0KAzKkI5kAhvqYm2xeK4zyCbJ632j5i0b-M&dib_tag=se&keywords=Joystick+module+with+X%2FY+analog+outputs+and+push+button&nsdOptOutParam=true&qid=1775240336&sprefix=joystick+module+with+x%2Fy+analog+outputs+and+push+button%2Caps%2C177&sr=8-2)
- Jumper wires - [Buy On Amazon](https://www.amazon.com/sspa/click?ie=UTF8&spc=MTo2MTM3ODY2MDcxNzUxMzIyOjE3NzUyNDAyODI6c3BfYXRmOjIwMDAwNDcwNzU2NjI0MTo6MDo6&url=%2FElegoo-EL-CP-004-Multicolored-Breadboard-arduino%2Fdp%2FB01EV70C78%2Fref%3Dsr_1_1_sspa%3Fdib%3DeyJ2IjoiMSJ9.I3nSspk5onl8Jong0G-0EZ0Sm6UNwJPGx4KF6DIpVC5CCaibcMwpwsYDDnzrEc8Avt7r4hTUtPA044Kb7z39PYQrRDEibFnAkDchLapigCroiNShFMSXytk2VQ75fRKP47fAN0PGpIIHvlrio-J9yq6alQGBK2C7dxky3dL0-6WodX_YoZESkzJaZB3bQ2xrvX-2KEyB8ux7EUj61ns61S3yGK4JH8f_4H0s7ahheQk.i5s22XL93rBK4nsxhNvkSiOxxpHi16KF7S7YKRr049g%26dib_tag%3Dse%26keywords%3Djumper%2Bwires%26qid%3D1775240282%26sr%3D8-1-spons%26sp_csd%3Dd2lkZ2V0TmFtZT1zcF9hdGY%26psc%3D1)
- `LedControl` library installed from Arduino Library Manager - [Install Instructions](#step-1-install-the-library)

## Step 1: Install the library

1. Open the Arduino IDE.
2. Go to `Sketch` → `Include Library` → `Manage Libraries...`.
3. Search for `LedControl` by Eberhard Fahle.
4. Install the library.

Explanation:
- The `LedControl` library provides the functions needed to control the MAX7219 driver chip and draw pixels on the 8x8 LED matrix.
- Without this library, the Arduino sketch would need low-level SPI code to talk to the MAX7219, which makes the game much harder to write.

Try this:
- After installing the library, open the example sketch in Step 2 and run it to confirm your matrix is working.
- If you want a challenge, find the `LedControl` examples in the Arduino IDE and compare how `setRow()` and `setLed()` are used in this tutorial.

## Step 2: Wire the hardware

### LED matrix wiring

- `VCC` → 5V
- `GND` → GND
- `DIN` → Arduino pin `10`
- `CLK` → Arduino pin `13`
- `CS` / `LOAD` → Arduino pin `7`

Explanation:
- `DIN`, `CLK`, and `CS` are the three signals that talk to the MAX7219 driver.
- `VCC` and `GND` power the LED matrix, while `CS` tells the MAX7219 when a complete command is ready.
- This wiring makes the Arduino the controller, and the MAX7219 handles the LED refresh timing.

Try this:
- Draw a wiring diagram on paper showing the Arduino pins and the LED matrix connections.
- As a practice problem, swap the `DIN` and `CLK` wires and see what error message or behavior the matrix shows.

### Joystick wiring

- `VRx` → `A0`
- `VRy` → `A1`
- `SW` → digital pin `8`
- `GND` → GND
- `VCC` → 5V

### Optional LED indicator

- `ledPin` → digital pin `9`
- Use this only if you want a status LED for movement activity.

### Example: verify the matrix

Use this quick test sketch to confirm the MAX7219 matrix is wired correctly:

```cpp
#include <LedControl.h>

int DIN = 10;
int CLK = 13;
int CS  = 7;

LedControl lc = LedControl(DIN, CLK, CS, 1);

void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 5);
  lc.clearDisplay(0);
  for (int r = 0; r < 8; r++) {
    lc.setRow(0, r, 0xFF);
  }
}

void loop() {}
```

If the matrix lights up fully, the wiring is good. If not, check power, ground, and the `DIN`/`CLK`/`CS` pin connections.

## Step 3: Understand the sketch structure

The sketch is divided into these sections:

1. `#include <LedControl.h>` – load the matrix driver library
2. pin and game state setup
3. `setup()` to initialize the matrix and joystick pins
4. `loop()` to read the joystick and move the snake
5. helper functions for food, collisions, endgame, restart

Explanation:
- The sketch is organized so the hardware is set up once in `setup()`, and the game repeats in `loop()`.
- Game state variables hold the snake body, food location, direction, and whether the game is over.
- Helper functions keep the main loop easy to read by separating tasks like spawning food and showing the game over screen.

Try this:
- As a building exercise, write a one-sentence summary of what each section does.
- Then, identify which section of the code would need to change if you wanted to add a score counter.

## Step 4: Pin and game variables

Use these pin assignments in your sketch:

```cpp
int DIN = 10;
int CLK = 13;
int CS  = 7;

int xPin = A0;
int yPin = A1;
const int buttonPin = 8;
int ledPin = 9;

LedControl lc = LedControl(DIN, CLK, CS, 1);
```

Game state variables:

Explanation:
- `DIN`, `CLK`, and `CS` define the connection to the LED matrix.
- `xPin` and `yPin` read the joystick angles, while `buttonPin` detects restart presses.
- `snakeRow[]` and `snakeCol[]` store the snake body positions as arrays of coordinates.
- `direction` determines which way the snake will move on each update.

Try this:
- Change the initial snake position in the arrays and predict how the game start will look on the matrix.
- For a challenge, add a new `int score = 0;` variable and think where it should be updated when food is eaten.

```cpp
const int maxLength = 64;
int snakeLength = 3;
int snakeRow[maxLength] = {3, 3, 3};
int snakeCol[maxLength] = {3, 2, 1};
int row = 3;
int col = 3;
String direction = "right";

int foodRow = 0;
int foodCol = 0;

byte endgame[8] = {
  0x81, 0x42, 0x24, 0x18,
  0x18, 0x24, 0x42, 0x81
};

unsigned long lastMoveTime = 0;
int moveInterval = 300;
bool gameOver = false;
```

## Step 5: Initialize the display and inputs

In `setup()`, wake the matrix and configure the joystick pins.

```cpp
void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 1);
  lc.clearDisplay(0);

  pinMode(xPin, INPUT);
  pinMode(yPin, INPUT);
  pinMode(buttonPin, INPUT_PULLUP);

  randomSeed(analogRead(A2));
  spawnFood();
}
```

Explanation:
- `shutdown(0, false)` turns on the MAX7219 display.
- `setIntensity(0, 1)` sets a low brightness so the matrix is easy on the eyes.
- `INPUT_PULLUP` on the button pin makes the button read `HIGH` when idle and `LOW` when pressed.
- `randomSeed(analogRead(A2))` creates a better random pattern by reading a floating analog pin.

Try this:
- Add `Serial.begin(9600);` to `setup()` and print `xValue` and `yValue` from `loop()` to verify your joystick readings.
- As a task, change the initial brightness with `setIntensity()` and see how it affects the display.

Notes:

- `INPUT_PULLUP` is used for the joystick button so it reads `HIGH` when not pressed.
- `randomSeed(analogRead(A2))` adds randomness to food placement.

## Step 6: Read joystick input and choose a direction

The joystick values are read with `analogRead()`. The sketch uses thresholds to decide direction.

```cpp
int xValue = analogRead(xPin);
int yValue = analogRead(yPin);

if (xValue > 700 && direction != "left") direction = "right";
else if (xValue < 300 && direction != "right") direction = "left";
else if (yValue > 700 && direction != "up") direction = "down";
else if (yValue < 300 && direction != "down") direction = "up";
```

Explanation:
- The joystick gives two analog values, one for X and one for Y.
- The code compares those readings against high and low thresholds to decide whether the stick is being pushed in one of the four directions.
- The `direction != ...` rules prevent the snake from reversing into itself immediately.

Try this:
- Build a small debug sketch that prints `xValue` and `yValue` while you move the joystick; use it to confirm which range means left, right, up, or down.
- As a challenge, modify the code so the snake only changes direction after the joystick has been moved away from the center for 100 ms.

Tip: if your joystick values are different, adjust `700` and `300` to match the center and edge readings.

### Example: calibrate joystick values

Use this simple debug sketch to see the actual `analogRead()` values from your joystick:

```cpp
int xPin = A0;
int yPin = A1;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int xValue = analogRead(xPin);
  int yValue = analogRead(yPin);
  Serial.print("X=");
  Serial.print(xValue);
  Serial.print(" Y=");
  Serial.println(yValue);
  delay(200);
}
```

Move the joystick left, right, up, and down, then use the printed values to fine-tune the threshold numbers.

## Step 7: Move the snake without blocking

The sketch uses `millis()` to move the snake every `moveInterval` milliseconds.

```cpp
unsigned long currentTime = millis();
if (currentTime - lastMoveTime >= moveInterval) {
  lastMoveTime = currentTime;
  // update head position here
}
```

Explanation:
- `millis()` returns the number of milliseconds since the Arduino started.
- Checking the elapsed time instead of using `delay()` keeps the program able to read the joystick continuously.
- This is called non-blocking timing: the sketch only updates movement when enough time has passed.

Try this:
- Change `moveInterval` to `500` and `200` and observe how the game speed changes.
- For a practice problem, add a second speed mode that uses a button press to toggle between slow and fast movement.

This keeps the sketch responsive while reading joystick input continuously.

### Example: change the game speed

To make the snake faster or slower, adjust `moveInterval`:

```cpp
int moveInterval = 500; // slower
int moveInterval = 200; // faster
```

A larger value means slower movement, and a smaller value means faster movement.

## Step 8: Update the snake body

Before drawing, shift the body positions down the arrays so the new head can be inserted.

```cpp
for (int i = snakeLength - 1; i > 0; i--) {
  snakeRow[i] = snakeRow[i - 1];
  snakeCol[i] = snakeCol[i - 1];
}

snakeRow[0] = row;
snakeCol[0] = col;
```

Then draw the snake and food:

Explanation:
- The loop moves each segment to the position of the segment in front of it.
- Inserting the new head at index `0` makes the snake appear to move forward.
- The tail is effectively removed by overwriting its last position unless the snake has just eaten food.

Try this:
- Manually trace the arrays for one move: if the snake is at `[(3,3),(3,2),(3,1)]` and moves right, what should the arrays become?
- As a build exercise, modify the code so the snake starts with length 5 instead of 3.

```cpp
lc.clearDisplay(0);
for (int i = 0; i < snakeLength; i++) {
  lc.setLed(0, snakeRow[i], snakeCol[i], true);
}
lc.setLed(0, foodRow, foodCol, true);
```

## Step 9: Handle collisions

### Wall collision

If the head goes outside `0..7`, the game ends.

```cpp
if (col > 7 || col < 0 || row > 7 || row < 0) {
  showEndgame();
  return;
}
```

### Self collision

After the head moves, check if it hits any existing body segment.

```cpp
for (int i = 1; i < snakeLength; i++) {
  if (snakeRow[i] == row && snakeCol[i] == col) {
    showEndgame();
    return;
  }
}
```

### Food collision

If the head hits the food, grow the snake and place a new food item.

```cpp
if (row == foodRow && col == foodCol) {
  spawnFood();
  if (snakeLength < maxLength) snakeLength++;
}
```

Explanation:
- Wall collision checks whether the snake head left the 8x8 grid.
- Self collision loops through the body to see if the head overlaps any segment, including the tail.
- Food collision is detected when the head coordinates match the food coordinates, then the snake grows.

Try this:
- As a practice problem, change the wall collision to wrap around the edges instead of ending the game.
- Add a score variable that increases when food is eaten, then display it on the serial monitor.

## Step 10: Spawn food safely

The food is placed in a random free cell. If the snake already fills the board, the game ends.

```cpp
void spawnFood() {
  bool valid = false;
  int attempts = 0;

  while (!valid && attempts < 100) {
    foodRow = random(0, 8);
    foodCol = random(0, 8);
    valid = true;
    for (int i = 0; i < snakeLength; i++) {
      if (snakeRow[i] == foodRow && snakeCol[i] == foodCol) {
        valid = false;
        break;
      }
    }
    attempts++;
  }

  if (!valid) showEndgame();
}
```

Explanation:
- The function randomly chooses a row and column until it finds a cell the snake does not occupy.
- The `attempts` limit prevents an infinite loop if the board is almost full.
- If no free cell is found, the game ends because there is nowhere left to place food.

Try this:
- Write a version of `spawnFood()` that first collects all free cells into a list and then picks one at random.
- As a challenge, modify the function to avoid spawning food on the edges of the board.

## Step 11: Show game over and restart

When the player loses, the display shows an end pattern and waits for the button press.

```cpp
void showEndgame() {
  lc.clearDisplay(0);
  for (int i = 0; i < 8; i++) {
    lc.setRow(0, i, endgame[i]);
    delay(10);
  }
  delay(3000);
  lc.clearDisplay(0);
  gameOver = true;
}

void checkRestart() {
  if (digitalRead(buttonPin) == LOW) {
    delay(200);
    while (digitalRead(buttonPin) == LOW);
    resetGame();
    gameOver = false;
  }
}
```

Explanation:
- `showEndgame()` clears the display, draws a pattern, then pauses briefly before setting `gameOver`.
- `checkRestart()` waits for the button to be pressed, debounces it, and then calls `resetGame()`.
- The game stays frozen on the end screen until the player presses the button, then the snake is reset.

Try this:
- Add a `score` reset inside `resetGame()` and print the score to the serial monitor when the game restarts.
- For an extra challenge, make `showEndgame()` blink the display three times before showing the pattern.

Then reset the snake state:

```cpp
void resetGame() {
  snakeLength = 3;
  row = 3;
  col = 3;
  snakeRow[0] = 3; snakeCol[0] = 3;
  snakeRow[1] = 3; snakeCol[1] = 2;
  snakeRow[2] = 3; snakeCol[2] = 1;
  direction = "right";
  lastMoveTime = millis();
  spawnFood();
}
```

## Full sketch reference

Use the complete sketch below after you have verified wiring and library installation.

```cpp
#include <LedControl.h>

int DIN = 10;
int CLK = 13;
int CS  = 7;

int xPin = A0;
int yPin = A1;
const int buttonPin = 8;
int ledPin = 9;

LedControl lc = LedControl(DIN, CLK, CS, 1);

const int maxLength = 64;
int snakeLength = 3;
int snakeRow[maxLength] = {3, 3, 3};
int snakeCol[maxLength] = {3, 2, 1};
int row = 3;
int col = 3;
String direction = "right";

int foodRow = 0;
int foodCol = 0;

byte endgame[8] = {
  0x81, 0x42, 0x24, 0x18,
  0x18, 0x24, 0x42, 0x81
};

unsigned long lastMoveTime = 0;
int moveInterval = 300;
bool gameOver = false;

void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 1);
  lc.clearDisplay(0);

  pinMode(xPin, INPUT);
  pinMode(yPin, INPUT);
  pinMode(buttonPin, INPUT_PULLUP);

  randomSeed(analogRead(A2));
  spawnFood();
}

void loop() {
  if (gameOver) {
    checkRestart();
    return;
  }

  int xValue = analogRead(xPin);
  int yValue = analogRead(yPin);

  if (xValue > 700 && direction != "left") direction = "right";
  else if (xValue < 300 && direction != "right") direction = "left";
  else if (yValue > 700 && direction != "up") direction = "down";
  else if (yValue < 300 && direction != "down") direction = "up";

  unsigned long currentTime = millis();
  if (currentTime - lastMoveTime >= moveInterval) {
    lastMoveTime = currentTime;

    if (direction == "right") col++;
    else if (direction == "left") col--;
    else if (direction == "up") row--;
    else if (direction == "down") row++;

    if (col > 7 || col < 0 || row > 7 || row < 0) {
      showEndgame();
      return;
    }

    for (int i = snakeLength - 1; i > 0; i--) {
      snakeRow[i] = snakeRow[i - 1];
      snakeCol[i] = snakeCol[i - 1];
    }
    snakeRow[0] = row;
    snakeCol[0] = col;

    for (int i = 1; i < snakeLength; i++) {
      if (snakeRow[i] == row && snakeCol[i] == col) {
        showEndgame();
        return;
      }
    }

    if (row == foodRow && col == foodCol) {
      spawnFood();
      if (snakeLength < maxLength) snakeLength++;
    }

    lc.clearDisplay(0);
    for (int i = 0; i < snakeLength; i++) {
      lc.setLed(0, snakeRow[i], snakeCol[i], true);
    }
    lc.setLed(0, foodRow, foodCol, true);
  }
}

void spawnFood() {
  bool valid = false;
  int attempts = 0;

  while (!valid && attempts < 100) {
    foodRow = random(0, 8);
    foodCol = random(0, 8);
    valid = true;
    for (int i = 0; i < snakeLength; i++) {
      if (snakeRow[i] == foodRow && snakeCol[i] == foodCol) {
        valid = false;
        break;
      }
    }
    attempts++;
  }

  if (!valid) showEndgame();
}

void showEndgame() {
  lc.clearDisplay(0);
  for (int i = 0; i < 8; i++) {
    lc.setRow(0, i, endgame[i]);
    delay(10);
  }
  delay(3000);
  lc.clearDisplay(0);
  gameOver = true;
}

void checkRestart() {
  if (digitalRead(buttonPin) == LOW) {
    delay(200);
    while (digitalRead(buttonPin) == LOW);
    resetGame();
    gameOver = false;
  }
}

void resetGame() {
  snakeLength = 3;
  row = 3;
  col = 3;
  snakeRow[0] = 3; snakeCol[0] = 3;
  snakeRow[1] = 3; snakeCol[1] = 2;
  snakeRow[2] = 3; snakeCol[2] = 1;
  direction = "right";
  lastMoveTime = millis();
  spawnFood();
}

## Line-by-line explanation

- `#include <LedControl.h>` imports the LED matrix library.
- `int DIN = 10;` sets the data pin.
- `int CLK = 13;` sets the clock pin.
- `int CS = 7;` sets the chip select pin.
- `int xPin = A0;` sets the joystick X axis pin.
- `int yPin = A1;` sets the joystick Y axis pin.
- `const int buttonPin = 8;` sets the button input pin.
- `LedControl lc = LedControl(DIN, CLK, CS, 1);` creates the display object for one matrix.
- `const int maxLength = 64;` reserves space for every cell on the 8×8 board.
- `int snakeLength = 3;` starts the snake with three blocks.
- `int snakeRow[maxLength] = {3, 3, 3};` stores the row position for each snake segment.
- `int snakeCol[maxLength] = {3, 2, 1};` stores the column position for each snake segment.
- `int row = 3;` starts the head row at 3.
- `int col = 3;` starts the head column at 3.
- `String direction = "right";` sets the initial movement direction.
- `int foodRow = 0;` stores the food row coordinate.
- `int foodCol = 0;` stores the food column coordinate.
- `byte endgame[8] = { ... };` stores the game over pattern.
- `unsigned long lastMoveTime = 0;` keeps track of the last move time.
- `int moveInterval = 300;` sets how often the snake moves.
- `bool gameOver = false;` tracks whether the game is paused.

- `void setup()` initializes the display, joystick input, and starting food.
- `void loop()` repeats forever, updating the game or waiting for restart.
- `millis()` is used for timing so the game stays responsive.
- `digitalRead(buttonPin) == LOW` detects the button press because the pin is pulled low when pressed.
- `placeBlock()` tries to lock the moving block into the stack and checks if the move is valid.
- `render()` redraws the board each frame.
- `handleEnd()` displays win or loss feedback and freezes the game.
- `resetGame()` restores the initial state for another playthrough.

- `lc.shutdown(0, false);` turns on the MAX7219 display.
- `lc.setIntensity(0, 1);` sets the display brightness.
- `lc.clearDisplay(0);` clears all LEDs.
- `pinMode(xPin, INPUT);` and `pinMode(yPin, INPUT);` prepare the joystick pins.
- `pinMode(buttonPin, INPUT_PULLUP);` sets the button pin with internal pull-up.
- `randomSeed(analogRead(A2));` seeds random using analog noise.
- `spawnFood();` places the first food item.

- `if (gameOver) { checkRestart(); return; }` stops movement after a loss.
- `analogRead(xPin)` and `analogRead(yPin)` read joystick direction values.
- The direction checks update the snake direction without reversing into itself.
- `currentTime - lastMoveTime >= moveInterval` checks if it is time to move again.
- `row++`, `row--`, `col++`, `col--` move the snake head.
- The wall collision check ends the game when the head leaves `0..7`.
- The body shift loop moves each snake segment forward.
- The self-collision loop ends the game if the head hits the body.
- `spawnFood()` picks a new food position when the snake eats food.
- The draw loop lights the snake and food pixels.

- `spawnFood()` chooses a random free cell and avoids placing food on the snake.
- `showEndgame()` flashes the end pattern and sets `gameOver`.
- `checkRestart()` waits for a button press and resets the game.
- `resetGame()` restores the starting snake state and spawns new food.

## Common problems and fixes

- **Matrix does not light up**
  - Verify the display is powered with 5V and GND.
  - Check `DIN`, `CLK`, `CS` wiring.
  - Make sure `LedControl` is installed.

- **Snake does not move**
  - Confirm joystick analog pins are connected to `A0` and `A1`.
  - Use `Serial.println(xValue)` / `Serial.println(yValue)` to see values.
  - Adjust thresholds if your joystick center is not near `512`.

- **Joystick button restart doesn’t work**
  - `buttonPin` must be `INPUT_PULLUP`.
  - The button should connect pin `8` to GND when pressed.
  - If it reads backwards, swap the wiring or invert the logic.

- **Food appears on the snake**
  - The `spawnFood()` loop checks all snake segments before accepting the position.
  - If the snake fills most of the board, the code ends the game after 100 attempts.

- **Game freezes after loss**
  - `showEndgame()` sets `gameOver = true` and waits for the button.
  - If the button is noisy, the debounce `delay(200)` helps.

## To do and not to do

### Do

- Do test the matrix with a simple `LedControl` example first.
- Do wire the joystick button with pull-up logic.
- Do use `millis()` timing instead of `delay()` in the main game loop.
- Do keep the head and body arrays in sync.
- Do adjust `moveInterval` to make the game faster or slower.

### Don’t

- Don’t use `delay()` inside `loop()` except for endgame or debouncing.
- Don’t hardcode `numDevices` incorrectly if you have only one matrix.
- Don’t let the snake reverse direction immediately into itself.
- Don’t assume the matrix orientation is correct; rotate rows/columns if needed.

## Tips

- If the display is rotated, swap `row` and `col` or rotate the matrix pattern.
- Use a second sketch with `Serial.println()` to calibrate joystick edges.
- If the game is too hard, increase `moveInterval`.
- If the snake moves too slowly, decrease `moveInterval`.

---

Back to [index](index.md)
