# Heavy-Duty UAV Avionics & Power Interconnect Architecture

### System Electrical Specifications
* **Primary Battery:** 12S LiPo / Li-ion (44.4V Nominal, 50.4V Peak), 22,000 mAh, 25C continuous.
* **Peak Traction Current:** 180A (45A per arm at 100% thrust).
* **Hover Traction Current:** 52A (13A per arm at ~50% throttle).
* **Regulated Avionics Rails:** 
  * 5.3V @ 3A (Primary Flight Controller Power Module)
  * 12.0V @ 5A Step-Down Buck (Companion Computer & FPV/Payload)
  * 5.0V @ 2A Isolated Bus (RTK-GNSS, Telemetry Transceiver, Optical Flow)

---

### Subsystem Interconnect & Signal Routing Matrix

| Subsystem | Source Component | Destination / Controller | Interface / Protocol | Wire Gauge (AWG) | Pinout / Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Main Power Rail** | 12S LiPo Battery | Mauch/Power Distribution Board | Raw Battery (44.4V) | 8 AWG High-Flex Silicone | Anti-spark QS8 / AS150 Connector |
| **Motor Drive 1-4** | Power Distribution Board | 4x 60A–80A Opto ESCs | Raw DC Battery Rail | 12 AWG | Shielded twisted pair |
| **ESC Telemetry/PWM**| Flight Controller (MAIN 1-4) | 4x ESC Signal Inputs | DShot600 / Bi-directional DShot | 26 AWG Twisted Pair | Signal + Ground referenced |
| **Primary GNSS / Mag**| Here4 / Dual RTK GNSS | FC GPS1 Port | CAN Bus (DroneCAN) | 28 AWG Twisted 4-core | 120Ω terminating resistor enabled |
| **Secondary Telemetry**| RFD900x Long-Range Modems | FC TELEM1 Port | UART (57600 baud, CTS/RTS) | 28 AWG | 5V, GND, TX, RX, CTS, RTS |
| **Companion Compute**| FC TELEM2 / USB-CDC | Jetson Orin Nano / RPi | MAVLink (921600 baud) | 26 AWG Shielded USB | High-rate offboard telemetry |
| **Payload Rail** | 12V Buck Converter | Servo/Gimbal Trigger | PWM / GPIO (Aux 1-2) | 22 AWG | Regulated isolated ground loop |

---

### Grounding & EMI Isolation Strategy
1. **Star-Ground Topology:** All high-current motor ground returns route back to the central PDB negative terminal to eliminate ground bounce on sensitive ADC telemetry channels.
2. **Signal Shielding:** CAN bus and telemetry serial cables utilize twisted pairs with single-ended outer ground shields grounded solely at the Flight Controller chassis ground.
3. **Optocoupled Isolation:** ESC control lines utilize optocouplers to prevent transient motor back-EMF spikes from breaching the logic board.
