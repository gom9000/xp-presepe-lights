# Presepe lights-controller eXPerience
This laboratory experience documents the design and development of a modular control unit for the lighting of a crib (nativity scene).

The core challenge of this project is to build a complete scenery controller following a strict analog philosophy: no microcontrollers, no firmware, and no digital ICs. Everything is implemented using exclusively discrete components, operational amplifiers, and classic analog design techniques.
The goal is to create a system that can simulate the continuous phases of a day (sunrise, day, sunset, night) with smooth fade-in and fade-out transitions, flickering fire effects, and shimmering stars, while remaining completely immune to supply voltage variations between $12\text{VDC}$ and $24\text{VDC}$.

![overview](resources/presepe-controller.jpg)

## Specifications
* **Pure Analog Topology:** Timings, thresholds, and waveshapes are derived entirely via discrete transistors, diodes, and op-amps.
* **Supply Voltage Immunity:** The choice of the supply voltage ($V_{CC}$) must not affect the internal timing constants or comparator thresholds. It should only determine the maximum measurable Zener voltage or the maximum length of the LED strings.
* **Scenic Effects:**
  * Full day-cycle simulation split into four distinct phases: *Sunrise, Day, Sunset, Night*.
  * Independent, adjustable trapezoidal fade-in/fade-out current loops for output lines.
  * Scintillating star tremolo and chaotic firelight emulation.
* **Modular Architecture:** The system is split into independent functional blocks that communicate through a common passive backplane BUS, allowing for easy troubleshooting and hot-swappable boards.

## Logical Modules

### Sawtooth wave generator
The heartbeat of the entire system is a sawtooth signal that rises linearly over a very long period, representing the duration of a full day in the presepe.

To produce this ramp, a constant current generator linearly charges a high-capacity master capacitor. Once the voltage reaches its peak, a comparator with hysteresis (Schmitt trigger) detects the threshold and rapidly activates a discharge network to reset the ramp.

**Design Note:** When adapting transistor biasing to the correct voltage levels, care must be taken never to exceed their maximum ratings. For instance, the base-emitter reverse voltage must be kept safely below the datasheet $V_{eb0}$ breakdown limit (typically around $4\text{V} - 5\text{V}$).

![image](resources/presepe-sawtooth.jpg)

### Daily-phase triggers generator
The linear master ramp is split it into logical sub-periods. By feeding the sawtooth signal into four parallel voltage window-comparators, we can slice the continuous ramp into four distinct timeframes. This allows us to obtain four High/Low trigger signals representing the related phases: sunrise ($t_A$), day ($t_G$), sunset ($t_T$), and night ($t_N$).

![image](resources/presepe-trigger.jpg)

### LFO square wave generator
To simulate the rapid twinkling of starlight, a faster modulation source is used. This is achieved through a standard free-running astable multivibrator tuned to act as a low-frequency oscillator (around $15\text{Hz}$).

### The BUS
To maintain modularity, all power lines, triggers, and wave signals travel across a shared backplane. Signals on the BUS are actively buffered to increase fan-out and prevent loading effects between boards. The BUS layout consists of:
* **Power Rails:** $+V_{CC}$, $\text{GND}$
* **Phase Triggers:** $t_A$ (Sunrise), $t_G$ (Day), $t_T$ (Sunset), $t_N$ (Night)
* **Analog Waveforms:** $S$ (Master Sawtooth), $lfo$ ($15\text{Hz}$ Modulation Wave)

### Daily-Phase lights fade in/out effects
To avoid harsh steps when a phase turns on or off, we need to gradually dim the LED strings.

The circuit uses a constant-current generator to charge a timing capacitor, with the charge cycle controlled by a voltage comparator driven by the phase-trigger signal. When the trigger is deactivated, a diode network switches the path, discharging the capacitor into a dedicated current-sink generator. The rectangular trigger input signal is now a beautifully smooth, trapezoid-shaped voltage signal.

![image](resources/presepe-trapezoidal.jpg)

### Triggers Mixer
To force a light channel to stay active across multiple compound phases (like sunrise + day, or day + sunset), mixing the triggers without cross-talk, the circuit implements a simple diode OR-gate network, followed by a passive level-shifter stage to clamp the low-signal state tightly back to physical $\text{GND}$.

![image](resources/presepe-tmixer.jpg)

### Voltage to current converter
Since LEDs are current-driven devices, an emitter-follower stage combined with discrete BJT current mirrors serves as both a linear voltage-to-current converter and a multi-line output doubler. This gracefully transforms the synthesized trapezoidal voltage waves into proportional current loops.

![image](resources/presepe-mirrors.jpg)

### Star-light effect
By summing the high-frequency $lfo$ wave onto the slow-moving trapezoidal phase voltage, the output driver receives a serrated control signal that emulates the tremolo of the stars light.

### Fire-light effect
(Development ongoing — working on a discrete chaotic relaxation oscillator network to emulate the random tremolo of fire embers).


## Physical Boards
The modules are deployed across these physical hardware blocks:
- [waves-board](waves-board): Implements the master sawtooth wave generator and the LFO circuit.
- [triggers-board](triggers-board): Contains the quad window-comparator array.
- [backplane-board](backplane-board): A passive motherboard with 7 slots to route the distribution BUS.
- [phases-lines-board](phases-lines-board): Implements quad current-driven output lines with independent fade tuning and routing.
- Fire-lights Lines Board: *(In development)*


## About & Credits
* **Author:** Alessandro Fraschetti (gom9000)
* **Special Thanks:** Alan Wolke (*w2aew*) for his fantastic [tutorials](https://www.youtube.com/channel/UCiqd3GLTluk2s_IBt7p_LjA) that heavily inspired several discrete building blocks of this project.
* **License:** This eXPerience documentation and all associated hardware layouts are open-source under the [MIT License](LICENSE).
