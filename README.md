# The Wall of Soundlight: A Mixed-Signal Analog Audio Analyzer

**The Wall of Soundlight** is an advanced mixed-signal engineering project that physically separates and visualizes the human voice frequency spectrum. 

This repository documents the evolution of the hardware architecture—from discrete through-hole breadboard validation to a 4-layer PCB design with isolated ground and power planes.

## Phase 1: Analog Signal Processing & Proof of Concept

Before moving to the complexities of a 4-layer PCB or DSP code, the core audio processing architecture was physically validated on interconnected breadboards. This phase proved our mathematical models and demonstrated that we could isolate distinct frequency bands and drive high-power LEDs without digital microcontrollers.

### 🔬 Architecture & Signal Chain
The breadboard prototype implemented the following discrete stages using Op-Amps (NE5532, LM358, TL084):

1. **Preamplifier Stage:** Initial signal conditioning to boost the raw electret microphone input up to a usable voltage rail.
2. **Active Filtering (Spectrum Splitting):** Designed and tuned 3rd and 4th-order Sallen-Key bandpass filters. These separated the audio into Low, Mid, and High-frequency bands.
3. **Precision Rectifier (Envelope Follower):** Converted the AC audio waveforms into a dynamic DC control voltage, closely tracking the music's peaks and decays without diode voltage drop loss.
4. **Custom Analog PWM Generation:** Instead of digital PWM, we built a relaxation oscillator to generate a constant 6 kHz triangular carrier wave. We fed this carrier and the audio envelope into high-speed comparators to generate a highly responsive PWM signal through voltage intersection. 

### 🛠️ Hardware Debugging & Simulation
This phase wasn't just wiring—it was a battle against physics. We encountered and resolved real-world signal integrity issues:
- **Simulation Validation:** Modeled the filter curves (`FILTROS Cuarto.asc`) in LTSpice to ensure the physical resistors and capacitors matched our theoretical crossover frequencies.
- **Grounding & Parasitics:** Overcame the inherent parasitic capacitance of breadboards and grounding loops that caused signal distortion.
- **Power Isolation Testing:** Validated the absolute necessity of a split power topology. We proved that driving LEDs directly from the audio rails induced severe switching noise, cementing the requirement for a dedicated differential rail (±12V) isolated from the sensitive audio path (±10V) in the final PCB.

<img width="4160" height="1874" alt="WhatsApp Image 2026-09-26 at 1 32 51 PM" src="https://github.com/user-attachments/assets/f2448aa0-eb5c-4906-9201-cb430a57ce84" />

<img width="1200" height="1600" alt="WhatsApp Image 2026-09-26 at 1 32 51 PM (6)" src="https://github.com/user-attachments/assets/5ca402a7-d4ec-49e7-af66-eed655b3272a" />

Phase 2: Digital Signal Processing (DSP) via Teensy 4.0
To evaluate the trade-offs between a purely analog hardware approach and a software-defined architecture, we developed a parallel digital prototype. This phase replaced the physical Sallen-Key filters with a high-performance microcontroller executing DSP algorithms.

💻 System Architecture
Processing Core: Teensy 4.0 (Cortex-M7 at 600 MHz) utilizing the PJRC Audio Library for real-time DSP.

Digital Filtering: Programmed internal Biquad IIR filters to segment the audio into Low (80-250 Hz), Mid (250-550 Hz), and High (550-1250 Hz) bands.

Signal Reconstruction: Implemented high-speed SPI communication (at 20 MHz) to drive three external 12-bit DACs (MCP4921) simultaneously, pushing the processed digital envelopes back into the analog domain.

🔧 Engineering Challenges & Troubleshooting
Working across the digital-analog boundary introduced critical signal integrity challenges that validated our eventual move to a custom PCB:

Quantization Noise (The "Staircase" Effect): The DAC's discrete voltage steps introduced high-frequency ultrasonic noise into the output. We diagnosed this via oscilloscope and resolved it by implementing 2nd-order analog Butterworth reconstruction filters.

SPI Bus Signal Degradation: Driving three DACs on a breadboard via "Daisy Chain" caused severe pulse deformation at 20 MHz due to parasitic inductance/capacitance.

DC Bias Correction: The DAC output contained a 1.65V DC offset. We designed a capacitive coupling stage (10µF) to block the DC component and re-center the audio waveform at true 0V.

Software Calibration: Implemented adjustable software sensitivity multipliers (virtual amplifiers) and noise gates (noiseThreshold) to dynamically adjust the LED response without requiring hardware gain resistor changes.
