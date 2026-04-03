# Stacker

[← Back to index](index.md)

## Overview

This tutorial shows how to build a Stacker game on an 8x8 LED matrix using the `LedControl` library and a single button.

You will learn:

- how to wire the MAX7219 matrix and button
- how the Arduino sketch moves the block automatically
- how stacking logic, collision checking, and win/loss handling work
- how to reset the game after a win or loss

<details>
<summary><strong>Jump to section</strong></summary>

- [What you need](#what-you-need)
- [Step 1: Install the library](#step-1-install-the-library)
- [Step 2: Wire the hardware](#step-2-wire-the-hardware)
- [Step 3: Understand the game flow](#step-3-understand-the-game-flow)
- [Step 4: Pin and game variables](#step-4-pin-and-game-variables)
- [Step 5: Initialize the display and inputs](#step-5-initialize-the-display-and-inputs)
- [Step 6: Move the block automatically](#step-6-move-the-block-automatically)
- [Step 7: Handle button press to stack the block](#step-7-handle-button-press-to-stack-the-block)
- [Step 8: Place the block and check alignment](#step-8-place-the-block-and-check-alignment)
- [Step 9: Draw the stack and moving block](#step-9-draw-the-stack-and-moving-block)
- [Step 10: Handle win and loss conditions](#step-10-handle-win-and-loss-conditions)
- [Step 11: Reset the game](#step-11-reset-the-game)
- [Line-by-line explanation](#line-by-line-explanation)
- [Full sketch reference](#full-sketch-reference)
- [Common problems and fixes](#common-problems-and-fixes)

</details>

## What you need

- Arduino board (Uno, Nano, etc.) - [Buy On Adafruit](https://www.adafruit.com/product/2488)
- 9V battery clip with 5.5mm/2.1mm plug - [Buy On Adafruit](https://www.adafruit.com/product/80?gad_source=1&gad_campaignid=23438252138&gbraid=0AAAAADx9JvSzl8Zwe_PvvlEqgxzDMWtat&gclid=Cj0KCQjwyr3OBhD0ARIsALlo-OlHya_VpHYPBQZNaTq0mcxw_bNJ4wg3DxYBWZ-KisDtH-hRyNIa5qQaApWfEALw_wcB)
- 9V Battery - [Buy On Adafruit](https://www.google.com/aclk?sa=L&ai=DChsSEwi-1s2fptKTAxW_wp8JHRC7KYkYACICCAEQBhoCbWQ&co=1&ase=2&gclid=Cj0KCQjwyr3OBhD0ARIsALlo-OkZ5bcJ2kaCJucQ7CM-WFjatWCXsCKQF5kVA9DIs0tspzK4S_BgYu4aAoEzEALw_wcB&cid=CAASugHkaHZYuCchP4cNnNJ5WnR2QJBqcVmrQKg9tWa1grMBDa6SJmBY2A8jk1PlX__p3rVaXNlEbtwlEIESDmbCca-07XV9WP13-do2hn9VwFUHJM2hmEplgmJNlNGe9nZEJxf56PlCsB0p2UobYDEQiKuqDD_5C6vzf8k69tH7ViptjU095fkiaHFXBvDXOBp7fcySnCXQphcdLEPCOrM0vU2KPXXnzTXLNsA-rAl_mtIy-NNTMXoa9NY54ls&cce=2&category=acrcp_v1_32&sig=AOD64_2ut9HYXtbhVhvZ3VmjfJIuazjS2A&ctype=5&q=&nis=4&ved=2ahUKEwja88efptKTAxXprYkEHe4oI-IQ5bgDKAB6BAgIEAs&adurl=)
- 8x8 LED matrix with MAX7219 driver - [Buy On Amazon](https://www.amazon.com/sspa/click?ie=UTF8&spc=MTozNTQzMDcxNDE1MTQ3MTA2OjE3NzUyNDA0MjE6c3BfYXRmOjIwMDA5MTE4ODU4MTcxMTo6MDo6&url=%2FHiLetgo-MAX7219-Matrix-Display-Control%2Fdp%2FB07W6KZR5D%2Fref%3Dsr_1_1_sspa%3Fcrid%3D2WA7SUKVVS82N%26dib%3DeyJ2IjoiMSJ9.ZyHxd8WbxVSm4mZYyszaNNNuhcoATQLIWdgXpkkFx2H8lUMvbg0SnpUf9U77BbpRvwmPY2Oooi8-TCaB5--aFIopTz0GN3Dw6ciZI2r5Dh790ZXCEbi4kggajJFhmfQMarZLEzHlH3gJUKD_SYBVBM4h3otrhYvgoCECdoVrXxP0I0TrhmrwNcSkVdTrRB9UNtTdLBgcp4msjAYzWhPxoHoNhX7QYt-KBmB9RXFVJrltlOzmqbC1hSJbyLw_D0haxWeN3ZY1WuXN2Lw-66nHAm3hbzlq1FIIIkHOiGtBTvk.hPjjj95xfUkdNns58r2X1JGYcy3SH-uM4u3KKIfmK8Y%26dib_tag%3Dse%26keywords%3D8x8%2Bled%2Bmatrix%26qid%3D1775240421%26sprefix%3D8x8%2Bled%2Bm%252Caps%252C162%26sr%3D8-1-spons%26sp_csd%3Dd2lkZ2V0TmFtZT1zcF9hdGY%26psc%3D1)
- Arcade Button - [Buy On Adafruit](https://www.adafruit.com/product/3491)
- Jumper wires - [Buy On Amazon](https://www.amazon.com/sspa/click?ie=UTF8&spc=MTo2MTM3ODY2MDcxNzUxMzIyOjE3NzUyNDAyODI6c3BfYXRmOjIwMDAwNDcwNzU2NjI0MTo6MDo6&url=%2FElegoo-EL-CP-004-Multicolored-Breadboard-arduino%2Fdp%2FB01EV70C78%2Fref%3Dsr_1_1_sspa%3Fdib%3DeyJ2IjoiMSJ9.I3nSspk5onl8Jong0G-0EZ0Sm6UNwJPGx4KF6DIpVC5CCaibcMwpwsYDDnzrEc8Avt7r4hTUtPA044Kb7z39PYQrRDEibFnAkDchLapigCroiNShFMSXytk2VQ75fRKP47fAN0PGpIIHvlrio-J9yq6alQGBK2C7dxky3dL0-6WodX_YoZESkzJaZB3bQ2xrvX-2KEyB8ux7EUj61ns61S3yGK4JH8f_4H0s7ahheQk.i5s22XL93rBK4nsxhNvkSiOxxpHi16KF7S7YKRr049g%26dib_tag%3Dse%26keywords%3Djumper%2Bwires%26qid%3D1775240282%26sr%3D8-1-spons%26sp_csd%3Dd2lkZ2V0TmFtZT1zcF9hdGY%26psc%3D1)
- `LedControl` library installed from Arduino Library Manager - [Install Instructions](#step-1-install-the-library)


## Step 1: Install the library

1. Open the Arduino IDE.
2. Go to `Sketch` → `Include Library` → `Manage Libraries...`.
3. Search for `LedControl` by Eberhard Fahle.
4. Install the library.

Explanation:
- `LedControl` gives you functions to talk to the MAX7219 chip and draw pixels on the matrix.
- Without it, you would need to write your own SPI control for the display.

Try this:
- Open the `LedControl` library examples and compare `setLed()` with `setRow()`.

## Step 2: Wire the hardware

### LED matrix wiring

- `VCC` → 5V
- `GND` → GND
- `DIN` → Arduino pin `10`
- `CLK` → Arduino pin `13`
- `CS` / `LOAD` → Arduino pin `7`

### Button wiring

- one side of the button → digital pin `8`
- other side of the button → GND
- use `INPUT_PULLUP` in code so the button reads `HIGH` when not pressed and `LOW` when pressed

Explanation:
- The matrix uses `DIN`, `CLK`, and `CS` to receive commands from the Arduino.
- The button uses the Arduino's internal pull-up resistor, which eliminates the need for an external resistor.

Try this:
- Draw a wiring diagram showing the matrix and button connections.
- Test the button in a simple sketch that prints `LOW` when pressed.

## Step 3: Understand the game flow

The sketch has three main parts:

1. `setup()` initializes the display, button, and game state.
2. `loop()` moves the current block, checks for button presses, and ends or resets the game.
3. helper functions handle stacking, rendering, game end, and reset.

Explanation:
- `setup()` runs once at startup.
- `loop()` repeats forever, so the game is always updating.
- Helpers keep the main loop readable and make each behavior easier to understand.

Try this:
- Write a short description of what `placeBlock()` does in one sentence.
- Identify which function you would change if you wanted to add a score display.

## Step 4: Pin and game variables

Use these pin assignments in your sketch:

```cpp
int DIN = 10;
int CLK = 13;
int CS  = 7;
const int buttonPin = 8;

LedControl lc = LedControl(DIN, CLK, CS, 1);
```

Game variables:

```cpp
int currentRow = 7;      // Start at the bottom
int currentWidth = 3;    // Start with 3 blocks
int currentPos = 0;      // Leftmost position of the moving block
int direction = 1;       // 1 for Right, -1 for Left
bool stack[8][8];        // Memory of placed blocks

unsigned long lastMoveTime = 0;
int moveInterval = 200;  // Speed of the moving block
bool gameOver = false;
```

Explanation:
- `currentRow` is the row where the moving block currently travels.
- `currentWidth` is how many consecutive LEDs are lit for the moving block.
- `currentPos` is the leftmost column position of that moving block.
- `direction` controls whether the block moves right or left.
- `stack[8][8]` remembers which cells on the board are already stacked.
- `moveInterval` controls how fast the moving block updates.
- `gameOver` stops the normal game loop after a win or loss.

Try this:
- Change `currentWidth` to `4` and imagine how the first moving block looks.
- As a challenge, add a `score` variable that grows each time a block is placed successfully.

## Step 5: Initialize the display and inputs

```cpp
void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 5);
  lc.clearDisplay(0);

  pinMode(buttonPin, INPUT_PULLUP);
  
  // Initialize stack as empty
  for (int r = 0; r < 8; r++) {
    for (int c = 0; c < 8; c++) stack[r][c] = false;
  }
}
```

Explanation:
- `shutdown(0, false)` powers on the display.
- `setIntensity(0, 5)` sets a medium brightness so the block is visible.
- `clearDisplay(0)` turns off every LED.
- `pinMode(buttonPin, INPUT_PULLUP)` makes the button stable with internal pull-up.
- The nested for-loops set every cell in `stack` to `false`, clearing the game memory.

Try this:
- Add `Serial.begin(9600);` and print a message from `setup()` to confirm the board started.
- As a task, change the intensity value and note how the output brightness changes.

## Step 6: Move the block automatically

```cpp
unsigned long currentTime = millis();
if (currentTime - lastMoveTime >= moveInterval) {
  lastMoveTime = currentTime;
  
  currentPos += direction;
  
  // Bounce off walls
  if (currentPos + currentWidth > 8 || currentPos < 0) {
    direction *= -1;
    currentPos += (direction * 2); 
  }
  
  render();
}
```

Explanation:
- `millis()` gives the elapsed time since the Arduino started.
- The block only moves when the elapsed time reaches `moveInterval`.
- `currentPos += direction` slides the block left or right.
- The wall check reverses direction when the block hits the left or right edge.
- `render()` updates the display after the block moves.

Try this:
- Reduce `moveInterval` to `150` and see how much harder the game becomes.
- As a practice problem, modify the bounce logic so the block reverses one step earlier.

## Step 7: Handle button press to stack the block

```cpp
if (digitalRead(buttonPin) == LOW) {
  delay(200); // Debounce
  placeBlock();
  while(digitalRead(buttonPin) == LOW); // Wait for release
}
```

Explanation:
- `digitalRead(buttonPin) == LOW` detects the button press because the pin pulls down to ground when pressed.
- `delay(200)` debounces the button so one press doesn't register multiple times.
- `placeBlock()` attempts to lock the moving block into the stack.
- The `while` loop waits until the button is released before continuing.

Try this:
- Replace `delay(200)` with `delay(100)` and see if the button becomes more sensitive.
- For a challenge, add a second button to skip stacking and restart the level.

## Step 8: Place the block and check alignment

```cpp
void placeBlock() {
  int placedCount = 0;

  for (int i = 0; i < currentWidth; i++) {
    int col = currentPos + i;
    
    // If it's the bottom row, everything stays. 
    // Otherwise, check if there is a block underneath.
    if (currentRow == 7 || stack[currentRow + 1][col]) {
      stack[currentRow][col] = true;
      placedCount++;
    }
  }

  // If no blocks landed on top of others, you lose
  if (placedCount == 0) {
    handleEnd(false);
    return;
  }

  currentWidth = placedCount; // Next row will be thinner
  currentRow--;               // Move up
  moveInterval -= 15;         // Speed up slightly

  if (currentRow < 0) {
    handleEnd(true);          // You reached the top!
  }
}
```

Explanation:
- `placedCount` counts how many ledges from the moving block successfully land.
- The loop checks each LED in the current moving block.
- On the bottom row (`currentRow == 7`), every block is valid.
- On higher rows, the block only stays if there is a block directly below it.
- If no part of the block lands correctly, the game ends.
- `currentWidth = placedCount` makes the next row narrower.
- `currentRow--` moves the next block up one row.
- `moveInterval -= 15` makes the game faster after each successful placement.
- If `currentRow < 0`, the player has stacked to the top and wins.

Try this:
- Make the game easier by slowing the speed increase: change `moveInterval -= 15` to `-10`.
- As a challenge, add a `missedBlocks` counter and display it via serial output.

## Step 9: Draw the stack and moving block

```cpp
void render() {
  lc.clearDisplay(0);

  // Draw the established stack
  for (int r = 0; r < 8; r++) {
    for (int c = 0; c < 8; c++) {
      if (stack[r][c]) lc.setLed(0, r, c, true);
    }
  }

  // Draw the currently moving block
  for (int i = 0; i < currentWidth; i++) {
    lc.setLed(0, currentRow, currentPos + i, true);
  }
}
```

Explanation:
- `clearDisplay(0)` removes the old frame before drawing the new one.
- The nested loops redraw every stored block in `stack`.
- The second loop draws the moving block at the current row and position.

Try this:
- Modify `render()` so the moving block is shown with different brightness than the stack.
- As a practice problem, change the code to draw the stack first and the moving block last, then explain why the order matters.

## Step 10: Handle win and loss conditions

```cpp
void handleEnd(bool win) {
  gameOver = true;
  for (int i = 0; i < 3; i++) {
    lc.clearDisplay(0);
    delay(200);
    if (win) {
      // Flash full screen for win
      for(int r=0; r<8; r++) lc.setRow(0, r, 0xFF);
    } else {
      // Simple X for loss
      lc.setRow(0, 0, 0x81); lc.setRow(0, 7, 0x81);
    }
    delay(200);
  }
}
```

Explanation:
- `handleEnd(true)` is called when the player reaches the top row.
- `handleEnd(false)` is called when the player misses the stack.
- The function flashes the display three times.
- For a win, the board lights up completely.
- For a loss, a simple X pattern appears.
- `gameOver = true` stops the normal `loop()` behavior.

Try this:
- Change the loss pattern from an X to a diagonal line.
- As a challenge, add a sound buzzer section that plays a tone when the player wins.

## Step 11: Reset the game

```cpp
void resetGame() {
  currentRow = 7;
  currentWidth = 3;
  currentPos = 0;
  moveInterval = 200;
  gameOver = false;
  for (int r = 0; r < 8; r++) {
    for (int c = 0; c < 8; c++) stack[r][c] = false;
  }
}
```

Explanation:
- `resetGame()` returns all game variables to their starting values.
- The stack memory is cleared so the next game starts fresh.
- The moving block begins again on the bottom row.

Try this:
- Add `Serial.println("Game reset");` so you can see when the game restarts.
- For a challenge, create a win counter that increases each time `resetGame()` is called after a win.

## Line-by-line explanation

- `#include <LedControl.h>` imports the LED matrix library.
- `int DIN = 10;` sets the data pin.
- `int CLK = 13;` sets the clock pin.
- `int CS = 7;` sets the chip select pin.
- `const int buttonPin = 8;` sets the button input pin.
- `LedControl lc = LedControl(DIN, CLK, CS, 1);` creates the display object for one matrix.
- `int currentRow = 7;` starts the moving block at the bottom row.
- `int currentWidth = 3;` starts the block three LEDs wide.
- `int currentPos = 0;` begins the moving block at the left edge.
- `int direction = 1;` makes the block move right initially.
- `bool stack[8][8];` stores which cells are already stacked.
- `unsigned long lastMoveTime = 0;` keeps track of the last movement update.
- `int moveInterval = 200;` sets how often the block moves.
- `bool gameOver = false;` tracks whether the game is paused.

- `void setup()` initializes the display, button input, and clears the stack memory.
- `void loop()` repeats forever and either updates the game or waits for a reset.
- `millis()` is used for timing so the game remains responsive.
- `digitalRead(buttonPin) == LOW` detects a button press because the pin is pulled low when pressed.
- `placeBlock()` tries to lock the moving block into the stack and checks if the move is valid.
- `render()` redraws the board each frame.
- `handleEnd()` displays win or loss feedback and freezes the game.
- `resetGame()` restores the initial state for another playthrough.

## Full sketch reference

Use the complete code below after you have verified wiring and library installation.

```cpp
#include <LedControl.h>

// Pin Definitions
int DIN = 10;
int CLK = 13;
int CS  = 7;
const int buttonPin = 8;

LedControl lc = LedControl(DIN, CLK, CS, 1);

// Game Variables
int currentRow = 7;      // Start at the bottom
int currentWidth = 3;    // Start with 3 blocks
int currentPos = 0;      // Leftmost position of the moving block
int direction = 1;       // 1 for Right, -1 for Left
bool stack[8][8];        // Memory of placed blocks

unsigned long lastMoveTime = 0;
int moveInterval = 200;  // Speed of the moving block
bool gameOver = false;

void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 5);
  lc.clearDisplay(0);

  pinMode(buttonPin, INPUT_PULLUP);
  
  // Initialize stack as empty
  for (int r = 0; r < 8; r++) {
    for (int c = 0; c < 8; c++) stack[r][c] = false;
  }
}

void loop() {
  if (gameOver) {
    if (digitalRead(buttonPin) == LOW) resetGame();
    return;
  }

  // 1. Handle Movement
  unsigned long currentTime = millis();
  if (currentTime - lastMoveTime >= moveInterval) {
    lastMoveTime = currentTime;
    
    currentPos += direction;
    
    // Bounce off walls
    if (currentPos + currentWidth > 8 || currentPos < 0) {
      direction *= -1;
      currentPos += (direction * 2); 
    }
    
    render();
  }

  // 2. Handle Button Press (Stacking)
  if (digitalRead(buttonPin) == LOW) {
    delay(200); // Debounce
    placeBlock();
    while(digitalRead(buttonPin) == LOW); // Wait for release
  }
}

void placeBlock() {
  int placedCount = 0;

  for (int i = 0; i < currentWidth; i++) {
    int col = currentPos + i;
    
    // If it's the bottom row, everything stays. 
    // Otherwise, check if there is a block underneath.
    if (currentRow == 7 || stack[currentRow + 1][col]) {
      stack[currentRow][col] = true;
      placedCount++;
    }
  }

  // If no blocks landed on top of others, you lose
  if (placedCount == 0) {
    handleEnd(false);
    return;
  }

  currentWidth = placedCount; // Next row will be thinner
  currentRow--;               // Move up
  moveInterval -= 15;         // Speed up slightly

  if (currentRow < 0) {
    handleEnd(true);          // You reached the top!
  }
}

void render() {
  lc.clearDisplay(0);

  // Draw the established stack
  for (int r = 0; r < 8; r++) {
    for (int c = 0; c < 8; c++) {
      if (stack[r][c]) lc.setLed(0, r, c, true);
    }
  }

  // Draw the currently moving block
  for (int i = 0; i < currentWidth; i++) {
    lc.setLed(0, currentRow, currentPos + i, true);
  }
}

void handleEnd(bool win) {
  gameOver = true;
  for (int i = 0; i < 3; i++) {
    lc.clearDisplay(0);
    delay(200);
    if (win) {
      // Flash full screen for win
      for(int r=0; r<8; r++) lc.setRow(0, r, 0xFF);
    } else {
      // Simple X for loss
      lc.setRow(0, 0, 0x81); lc.setRow(0, 7, 0x81);
    }
    delay(200);
  }
}

void resetGame() {
  currentRow = 7;
  currentWidth = 3;
  currentPos = 0;
  moveInterval = 200;
  gameOver = false;
  for (int r = 0; r < 8; r++) {
    for (int c = 0; c < 8; c++) stack[r][c] = false;
  }
}
```

## Common problems and fixes

- **The block does not move**
  - Confirm the MAX7219 wiring to `DIN`, `CLK`, and `CS`.
  - Make sure `pinMode(buttonPin, INPUT_PULLUP);` is set correctly.
  - If the block is frozen, check whether `gameOver` is stuck at `true`.

- **The block disappears at the edge**
  - The wall bounce check uses `currentPos + currentWidth > 8` and `currentPos < 0`.
  - If `currentWidth` is too large, the block may go out of bounds.

- **The button does not work**
  - The button should connect pin `8` to GND when pressed.
  - If it reads backwards, swap the wiring or use `INPUT_PULLDOWN` if your board supports it.

- **The stack is incorrect after placing a block**
  - Ensure `stack[currentRow][col] = true;` only runs when the block is above another block or on the bottom row.
  - If you change the loop boundaries, verify every column is checked correctly.

- **The game ends too quickly**
  - A narrow block or fast speed makes alignment harder.
  - Increase `moveInterval` or start with a larger `currentWidth`.

## Tips

- If the display looks inverted, try swapping row and column values in `setLed()`.
- Use `Serial.println(currentPos);` and `Serial.println(currentWidth);` to debug block position.
- If the game is too hard, increase `moveInterval` or start with a larger `currentWidth`.
- To make the game more forgiving, only shrink the block when there is a full overlap instead of partial overlap.

---

Back to [index](index.md)
