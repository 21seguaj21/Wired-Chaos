# Troubleshooting

[← Back to index](index.md)

## Common problems

### Device won't power on
- Check USB cable and power source
- Verify the board is seated correctly
- Ensure the voltage is correct for the LED strip/components
- Look for short circuits on the breadboard

### LEDs behave strangely
- Loose or incorrect wiring between Arduino, LEDs, and ground
- Wrong pin used in code
- Not enough current or missing resistor
- Using the wrong LED type in code (for example NeoPixel vs standard LED)

### Code won't upload
- Wrong board selected in the Arduino IDE
- Wrong COM port selected
- Driver not installed
- USB cable is power-only, not data-capable

### Game logic not working
- Variables not initialized
- Debounce missing for buttons
- Timing code conflicts with LED updates
- State machine logic not handling all cases

## Typical causes

- Breadboard connections shifted by one row
- Power supply sag when multiple LEDs turn on
- Using `delay()` inside routines that need responsive input
- Not calling `pinMode()` for buttons or outputs
- Pull-up/pull-down resistor issues on switches

## Debugging tips

- Test each component separately
- Use `Serial.println()` to trace code execution
- Check voltages with a multimeter
- Replace suspicious wires or pins
- Simplify the sketch to a minimal working example

## Intermittent problems

- Re-seat connections
- Check for heat or cold joints
- Confirm the board is not resetting due to power dips
- Look for shorts between adjacent wires

## When the project compiles but behaves incorrectly

- Verify hardware diagrams against your build
- Confirm the pin mapping in code matches the actual pins
- Use comments and step-by-step testing
- Watch for logic that depends on button press order

## Last resort

- Restart the Arduino IDE
- Disconnect and reconnect the board
- Reflash the bootloader only if the board is corrupted
- Try a simple "blink" sketch to verify the board itself works