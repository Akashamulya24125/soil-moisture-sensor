\# Soil Moisture Sensor — V1

\### Custom PCB Design · Capacitive Sensing · KiCad · Circuit Simulation



A custom soil-moisture sensing hardware project focused on PCB-level capacitive sensing, analog signal conditioning, circuit simulation, and hardware-to-software interface design.



This repository preserves the \*\*Soil Sensor V1 KiCad design\*\*, supporting design documentation, electrical-rule reports, and PCB fabrication outputs.



\---



\## 1. Project Overview



The objective is to develop a custom soil moisture sensor in which the sensing electrodes are formed directly by copper patterns on the PCB rather than using a separate commercial sensor breakout module.



The probe uses two interlocking copper comb electrodes separated by a small gap. Changes in the surrounding soil environment affect the effective capacitance of the sensing structure. The analog front end converts the resulting electrical response into an output voltage that can be measured by a microcontroller.



The project also defines an interface for reading sensor measurements through an Arduino Uno and displaying moisture readings on a laptop.



\*\*Primary engineering goals\*\*

\- Design the sensor circuitry and PCB using KiCad.

\- Implement a PCB-integrated capacitive sensing structure.

\- Simulate the oscillator and analog signal-conditioning stages.

\- Define a consistent electrical and serial communication interface.

\- Generate fabrication data and document the design for further testing.



\## 2. System Architecture



The documented system follows this signal path:



```text

Soil / Probe Environment

&#x20;         |

&#x20;         v

PCB Capacitive Comb Electrodes

&#x20;         |

&#x20;         v

555-Timer Oscillator

&#x20;         |

&#x20;         v

Sensing Network

&#x20;         |

&#x20;         v

Diode Rectifier + RC Filter

&#x20;         |

&#x20;         v

Analog Output (AOUT)

&#x20;         |

&#x20;         v

Arduino Uno — Analog Input A0

&#x20;         |

&#x20;         v

USB Serial Communication

&#x20;         |

&#x20;         v

Laptop Moisture Monitor Application

```



The custom sensor PCB provides the electrical signal. The Arduino reads the analog output, and the documented serial interface allows the laptop application to display the moisture percentage and raw ADC reading.



This describes the intended system architecture; physical end-to-end validation is still required.



\## 3. Hardware Design



\### PCB-integrated capacitive probe



The sensing element is formed by copper comb patterns in the PCB layout. The electrodes are not separate purchased components.



The soil occupies the electric-field region around and between the fingers. Changes in effective capacitance influence the analog sensing circuit.



Important design considerations include:

\- Electrode geometry and spacing.

\- Protection of the sensing structure from corrosion and unwanted exposure.

\- Separation of the sensing region from the electronics section.

\- Repeatable probe placement during calibration and testing.



\### Analog signal-conditioning circuit



The supplied schematic and circuit-design notes describe a 555-timer oscillator followed by a capacitive sensing network and a diode-based peak detector.



The circuit stages are:



1\. \*\*Oscillator:\*\* Generates the periodic excitation signal.

2\. \*\*Sensing network:\*\* Couples the probe's effective capacitance into the circuit response.

3\. \*\*Rectifier:\*\* Uses a small-signal diode to produce charging pulses.

4\. \*\*Filter and bleed network:\*\* Smooths the rectified signal into an analog output.

5\. \*\*Output interface:\*\* Provides VCC, GND, and AOUT connections.



The exact component packages and values should be checked against the current KiCad project and its committed BOM/footprint revision document.



\## 4. Electrical Interface



The accompanying Interface Control Document defines the intended Arduino-facing interface.



| Signal | Function |

|---|---|

| VCC | Sensor power input |

| GND | Common electrical reference |

| AOUT | Analog sensor output to Arduino A0 |



The documented prototype uses the Arduino Uno's 5 V supply and a common ground. The Arduino's 10-bit ADC produces raw readings in the range 0–1023.



The interface document gives an approximate AOUT expectation of \*\*1.5 V in wet conditions to 3.0 V in dry conditions\*\*. These are design expectations, not confirmed physical measurements.



\## 5. Circuit Simulation and Verification



The supplied simulation report documents electrical-only verification using \*\*KiCad 10 and ngspice\*\*.



\### Simulation configuration



| Parameter | Simulated setting |

|---|---|

| Supply voltage | 5 V DC |

| R1 | 1 kΩ |

| R2 | 10 kΩ |

| C1 | 1 nF |

| R3 | 10 kΩ |

| R4 | 100 kΩ |

| C3 | 10 nF |

| C4 | 100 nF |

| Probe representation | Ideal behavioral capacitance, C5 |

| Capacitance sweep | 100 pF–2.5 nF |

| Transient simulation end time | 50 ms |



\### Observed simulation results



The report records the following simulated behavior:



\- The 555 oscillator produced a repeating waveform at approximately 68.5 kHz.

\- The timing node exhibited the expected charge/discharge behavior.

\- Changes in the modeled sensing capacitance affected the Node-X waveform.

\- The diode exhibited repeated conduction events.

\- The RC network produced a smoother DC output at AOUT.

\- Increasing the modeled capacitance from 100 pF to 2.5 nF reduced the settled simulated AOUT voltage from approximately 2.90 V to 1.52 V.



These results demonstrate the expected behavior of the modeled signal chain under the tested simulation conditions.



\*\*Important distinction:\*\* C5 was an ideal simulated capacitor used to represent the sensing structure. It was not a measured capacitance of the physical PCB probe.



The simulated results therefore do not establish final moisture accuracy, real-soil performance, or the physical probe's dry/wet calibration.



\## 6. Arduino and Laptop Interface



The supplied firmware and application specifications define a wired monitoring architecture.



\### Arduino firmware interface



The firmware specification defines:

\- Analog input: Arduino A0.

\- ADC range: 0–1023.

\- Serial baud rate: 9600.

\- Update interval: 1000 ms, or one reading per second.

\- Output format:



```text

MOISTURE,<percent>,<raw>

```



Example:



```text

MOISTURE,62,743

```



The percentage is constrained to 0–100, while the raw field represents the direct ADC reading.



\### Laptop monitor application



The documented Python GUI is designed to:

\- Discover available serial/COM ports.

\- Connect to the Arduino.

\- Display the latest moisture percentage and raw ADC value.

\- Read serial data in a background thread so the GUI remains responsive.

\- Ignore malformed input lines.

\- Display a no-data status when valid readings have not arrived for more than three seconds.



The companion documents specify this integration contract. Executable firmware and laptop application source files are not included in the current tracked repository snapshot.



\## 7. Calibration and Physical Validation



Calibration values must be measured on the assembled sensor rather than inferred from simulation.



The supplied firmware document lists `DRY\_RAW = 750` and `WET\_RAW = 300` as \*\*placeholders\*\*. They must be replaced with actual measured values before final calibration.



The documented validation sequence is:



1\. Inspect the fabricated PCB and check continuity.

2\. Verify the supply and ground connections before connecting AOUT to the Arduino.

3\. Measure the output with the probe in open air under a controlled dry condition.

4\. Measure the output in the defined wet reference condition.

5\. Record the corresponding raw ADC readings.

6\. Update the firmware calibration constants using those measurements.

7\. Test the complete sensor → Arduino → USB serial → laptop application path.

8\. Evaluate repeatability under controlled soil conditions.



The supplied simulation report explicitly leaves physical probe validation and final dry/wet calibration pending.



\## 8. Repository Contents



The V1 repository contains the following project artifacts:



| File or directory | Purpose |

|---|---|

| `Soil\_Sensor\_v1.kicad\_pro` | KiCad project configuration |

| `Soil\_Sensor\_v1.kicad\_sch` | Circuit schematic |

| `Soil\_Sensor\_v1.kicad\_pcb` | PCB layout |

| `Soil\_Sensor\_v1.csv` | Project CSV data |

| `Soil\_Sensor\_SMD\_BOM\_Footprint\_Revision\_v1.2.pdf` | BOM and footprint revision documentation |

| `DRC.rpt` | Design Rules Checker report |

| `ERC.rpt` | Electrical Rules Checker report |

| `board2pdf.config.ini` | Board-to-PDF configuration |

| `plots/` | Gerber files, drill data, assembly PDF, and simulation-related files |

| `.gitignore` | Excludes selected local and temporary files |



The reports are included for review; their presence alone does not imply that every design rule or electrical check passed.



\## 9. Tools and Technologies



\- \*\*KiCad 10\*\* — schematic capture and PCB design.

\- \*\*ngspice\*\* — electrical circuit simulation.

\- \*\*Arduino Uno\*\* — intended analog measurement and serial interface.

\- \*\*Python / Tkinter / pySerial\*\* — documented laptop-monitor architecture.

\- \*\*Git and GitHub\*\* — version control and preservation of design history.



\## 10. Current Project Status



The V1 KiCad project and its documented fabrication outputs have been committed to Git and uploaded to GitHub.



The supplied simulation report records successful simulation checks of oscillator operation, capacitance-dependent response, rectification, and output smoothing.



The following are not established by those simulation results:

\- Final physical sensor calibration.

\- Accuracy across different soil types.

\- Repeatability of real-world measurements.

\- Final moisture-percentage performance of the assembled system.



These remain hardware-validation tasks rather than assumed achievements.



\## 11. Future Development



Future revisions can build upon the V1 design through documented, traceable changes to the schematic, PCB, sensing geometry, analog front end, calibration process, and system integration.



Each revision should preserve its design files and record its important changes so that previous versions remain available for comparison.



\---



\*\*Project:\*\* Soil Moisture Sensor — V1  

\*\*Design environment:\*\* KiCad 10  

\*\*Simulation environment:\*\* KiCad 10 + ngspice  

\*\*Repository purpose:\*\* Hardware design, documentation, simulation records, and fabrication data



