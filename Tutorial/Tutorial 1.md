# Build Log 01: Arduino + L298N Chassis Wiring

This document outlines the step-by-step process for wiring a 4-wheel drive robotics chassis using an Arduino and an L298N motor driver. It also includes troubleshooting steps for common power constraints and short-circuit issues encountered during the build.

## Prerequisites: Chassis Assembly & Motor Soldering

Depending on your kit, your 4WD chassis may arrive fully assembled, or you may need to build it from scratch. 

**Case A: Motors are Pre-wired**
If your four DC motors already have red and black wires attached, you only need to assemble the acrylic chassis. Refer to this tutorial to complete your chassis assembly:
* [How To Assemble 4WD Robot Smart Car Chassis Kits](https://www.youtube.com/watch?v=7ADyJrIgcpE&t=62s)

**Case B: Motors are Unwired (Soldering Required)**
If your DC motors have bare copper terminals, you will need to solder wires to them before assembling the chassis.
1. Take 8 jumper wires (male or female ends do not matter, as they will be clipped).
2. Cut one end off of each wire and strip the shielding until the bare copper is exposed.
3. Insert the exposed copper into the small holes on the DC motor's terminals (two wires per motor).
4. Solder the connections securely.
   * *Safety Note: Soldering irons operate at extreme temperatures. If you do not have steady hands or prior experience, ask a lab technician or an experienced peer for assistance.*
   * *Video Reference:* [Soldering Wires to a DC Motor](https://youtu.be/lCdlPS-6H6I?t=108)

Once the soldering is complete, return to the [Chassis Assembly Tutorial](https://www.youtube.com/watch?v=7ADyJrIgcpE&t=62s) to finish mounting the motors to the frame. 

Once your chassis is built, you can proceed with the logic and power wiring below.


## Hardware Components
* 4WD Robot Car Chassis Kit (with 4 DC Gear Motors)
* L298N Motor Driver Module
* Arduino Uno (or compatible)
* Jumper wires (Female-to-Male)
* Power supply (Minimum 7V required - e.g., 6x AA battery holder or 2x 18650 holder)

---

## Step 1: Wiring the Motors (Parallel Configuration)
Because the L298N only features two motor channels (A and B) but the chassis has four motors, the motors on each side must be wired in parallel to act as a single unit.

1. **Left Side:** 
   * Take one wire from the front-left motor and one from the back-left motor. Twist the exposed copper together. 
   * Repeat for the remaining two wires on the left side.
   * Insert one twisted pair into **OUT1** on the L298N and the other into **OUT2**.
2. **Right Side:**
   * Repeat the process for the right side, twisting the front and back wires into two pairs.
   * Insert one twisted pair into **OUT3** and the other into **OUT4**.

*Developer Note: Standard DC motors do not have strict polarity. If a wheel spins backwards during initial testing, simply unscrew those specific motor wires from the terminal, flip them, and re-secure them.*

---

## Step 2: Wiring the Logic (Arduino to L298N)
The Arduino acts as the controller, sending digital signals to the L298N to determine motor direction.

1. **Enable Caps:** Ensure the two small black jumper caps remain installed on the **ENA** and **ENB** pins (the far left and far right pins on the 6-pin logic block). This keeps the motor channels permanently enabled at full speed.
2. **Direction Pins:** Connect the middle four pins to the Arduino digital pins using female-to-male wires:
   * `IN1` -> Arduino Pin 8
   * `IN2` -> Arduino Pin 9
   * `IN3` -> Arduino Pin 10
   * `IN4` -> Arduino Pin 11
3. **Common Ground (CRITICAL):** Connect a wire from the Arduino's `GND` pin to the middle `GND` screw terminal on the L298N's 3-pin power block. The L298N and Arduino **must** share a ground connection to establish a common logic reference.

> **Tip:** Connect the motor driver logic wires directly to the Arduino headers rather than routing them through a breadboard. This ensures a secure connection and reserves breadboard space for 5V/GND sensor distribution (like ultrasonic or line trackers) later in the build.

---

## Step 3: Power Requirements & The 7V Rule
A common pitfall is attempting to power the L298N with a standard 3.6V or 5V battery pack (such as 3x AA batteries). 

The L298N features an internal voltage regulator that powers its logic circuitry and the red LED indicator. This regulator requires a **minimum of 7 Volts** to activate. Additionally, the driver itself drops about 2V as heat, leaving insufficient voltage for the motors if the input is too low.

**Recommended Power Setup:**
1. Utilize a battery pack capable of providing 7V to 12V.
2. Connect the Positive (Red) wire to the left-most **12V** terminal on the L298N.
3. Connect the Negative (Black) wire to the middle **GND** terminal (sharing it with the Arduino ground wire).
4. Leave the right-most **5V** terminal empty (unless specifically bypassing the regulator).

---

## Troubleshooting: USB Short Circuit
**Symptom:** Plugging the Arduino into a laptop via USB causes both the Arduino and L298N boards to light up briefly before the laptop instantly disables the USB port to protect itself.

**Diagnosis Process:**
To safely isolate the issue without risking laptop hardware, components were tested individually:
1. **Isolated the Arduino:** Unplugged the Arduino entirely from the L298N and connected it to the laptop. 
   * *Result: Arduino powered on normally. Laptop port remained active.*
2. **Isolated the L298N Logic:** Removed the battery supply from the L298N but kept the Arduino 5V, GND, and IN1-IN4 logic wires connected. 
   * *Result: Both boards powered on safely. This confirmed the logic wiring was not causing the short.*
3. **Identified the Culprit:** The short circuit only reproduced when the battery holder was physically wired into the L298N's power terminals. 

**Root Cause & Resolution:** 
The provided battery holder was internally shorted (positive and negative hardware making unintended contact inside the plastic casing). The faulty battery holder was discarded and replaced with a proper 9V (6x AA) unit, resolving the issue.

---

## Basic Test Code
Once the appropriate battery is installed and the red L298N indicator LED is illuminated, flash this basic sketch to verify all wheels drive forward.

```cpp
// Motor A (Left side)
const int IN1 = 8;
const int IN2 = 9;

// Motor B (Right side)
const int IN3 = 10;
const int IN4 = 11;

void setup() {
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
}

void loop() {
  // Command all motors to drive forward
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}
```
