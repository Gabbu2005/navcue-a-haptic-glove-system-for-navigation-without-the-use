# NavCue a haptic glove system for navigation without the use of eyes.

1. Executive Overview: The NavCue Board
Function: The NavCue PCB is a miniaturized, low-power wearable control board designed to be mounted on the dorsal (back) side of a riding glove. Its primary function is to act as a bidirectional Human-Machine Interface (HMI). It receives GPS navigation events via Bluetooth and translates them into spatial haptic feedback using six independent vibration motors. Simultaneously, it tracks hand movements, uses localized AI (TinyML) to detect deliberate gestures, and sends commands back to a smartphone.

Topology: The system uses a TWS (True Wireless Stereo) Master-Slave architecture. One glove's board acts as the "Master" (connecting to the phone and forwarding data), while the other acts as the "Slave" (connecting only to the Master glove). Both use the exact same PCB hardware.

2. Component-by-Component Functional Breakdown
A. The Core Logic & Communication (The "Brain")
Component: ESP32-C3-MINI-1 Module

Function: This is the central processing unit and wireless transceiver.

It maintains the Bluetooth Low Energy (BLE 5.0) connections between the phone and the other glove.

It runs the FreeRTOS firmware that manages all timing operations.

It executes the Edge Impulse TinyML neural network locally to classify gestures.

It generates the precise PWM (Pulse Width Modulation) signals required to create different vibration textures (e.g., a sharp pulse vs. a fading hum).

Why this specific module? It includes a built-in PCB antenna and an integrated metal EMI shield, which saves you from designing complex RF circuitry and ensures reliable Bluetooth range in an outdoor riding environment.

B. Motion Sensing (The "Senses")
Component: Bosch BMI323 (6-Axis IMU)

Function: This Inertial Measurement Unit contains a 3-axis accelerometer and a 3-axis gyroscope.

It continuously measures the glove's orientation, tilt, and acceleration in 3D space, sending this data to the ESP32-C3 via the I2C bus.

It allows the system to differentiate between normal riding vibrations (like engine rumble or bumps) and intentional rider gestures (like a wrist flick to change routes).

C. Haptic Actuation Subsystem (The "Muscles")
Components: 6x AO3400A N-Channel MOSFETs

Function: Microcontrollers cannot output enough current to drive a motor directly (an ESP32 outputs ~40mA; a coin motor needs ~70mA+). The MOSFET acts as an electronic switch. When the ESP32 sends a 3.3V logic signal to the MOSFET's "Gate", the MOSFET opens a channel that allows the raw battery current to flow through the haptic motor.

Components: 6x SS14 Flyback Diodes

Function: Motors are inductive loads. When you turn a motor off rapidly, the magnetic field collapses and generates a high-voltage spike that flows backward into the circuit. The flyback diode safely shunts this voltage spike away, preventing it from frying the MOSFET or the ESP32.

Components: JST-SH 2-Pin Connectors

Function: These low-profile surface-mount connectors allow the 6 haptic motors (which are stitched into the glove's fabric) to be plugged and unplugged from the main PCB easily, facilitating assembly and repairs.

D. Power Management & Delivery (The "Heart")
Component: USB Type-C Port (SMD)

Function: Provides a standard, reversible connection for 5V charging power and direct USB data transfer for programming the ESP32.

Component: MCP73831 (or TP4056) LiPo Charging IC

Function: Takes the 5V from the USB port and safely regulates it to charge the 3.7V Lithium-Polymer battery using a strict Constant-Current/Constant-Voltage (CC/CV) algorithm.

Component: DW01A + 8205A Dual MOSFET (Battery Protection)

Function: Often built into the battery cell itself (or placed on the PCB). It monitors the battery voltage. If the voltage drops too low (over-discharge) or spikes too high (over-charge), or if there is a short circuit, this chip instantly disconnects the battery to prevent a fire or battery death.

Component: AP2112K-3.3 (Low Dropout Regulator - LDO)

Function: The battery voltage fluctuates between 4.2V (fully charged) and 3.2V (empty). The ESP32 and IMU will burn out if given more than 3.6V. This LDO takes the varying battery voltage and steps it down to a perfectly flat, clean 3.3V.

E. User Interface & Peripherals
Component: External WS2812B LEDs (via 3-Pin JST Connector)

Function: Addressable RGB LEDs mounted on the back of the glove. They provide peripheral visual feedback to the rider (e.g., a green progress bar, an amber pairing light, or a red traffic warning) using just one data pin from the ESP32.

Components: Tactile Push Buttons (Reset & Pairing/Boot)

Function:

Reset Button: Briefly cuts power to the ESP32's Enable (EN) pin to reboot the system if it freezes.

Pairing/Boot Button: Connected to GPIO9. Held down during reset to put the board into programming mode. During normal operation, pressing this button triggers the Bluetooth pairing sequence to sync the Left and Right gloves together.

3. Critical Design Architecture Note: The "Split Power Rail"
One of the most important features of this board's design is the separation of power into two distinct "highways" (nets):

The Logic Rail (+3.3V): Powered by the LDO regulator. This only feeds the delicate components: the ESP32 and the BMI323 IMU.

The Raw Rail (VBAT): Sourced directly from the battery (bypassing the 3.3V regulator). This feeds the haptic motors and the LEDs.

Why is this important? When multiple haptic motors spin up simultaneously, they draw sudden spikes of current that can cause the voltage to briefly dip. By keeping the motors on the VBAT rail, these voltage dips do not affect the +3.3V rail, preventing the ESP32 from unexpectedly crashing or rebooting (brownout) in the middle of a ride.
