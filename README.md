# 🛡️ SentinelBridge — Emergency Operations Console & Radio-over-IP Voice Gateway

<div align="center">

[![SIH 2026](https://img.shields.io/badge/SIH%202026-Problem%20SIH26223-blue.svg?style=for-the-badge&logo=shield)](https://smartindiahackathon.gov.in/)
[![Theme](https://img.shields.io/badge/Theme-Disaster%20Management-red.svg?style=for-the-badge&logo=alert)](https://smartindiahackathon.gov.in/)
[![Category](https://img.shields.io/badge/Category-Hardware%20%2B%20AI%20Software-emerald.svg?style=for-the-badge)](https://smartindiahackathon.gov.in/)
[![Institution](https://img.shields.io/badge/Institution-NIT%20Silchar-purple.svg?style=for-the-badge)](http://www.nits.ac.in/)
[![License](https://img.shields.io/badge/License-Open%20Source-teal.svg?style=for-the-badge)](LICENSE)

**Team Aether_100 (Team ID: `NITS_100`) · National Institute of Technology Silchar**  
*Mentor: Dr. Koushik Guha · September 2026*

<br/>

[![Sentinel Bridge Overview & Hardware Poster](./image.png)](./image.png)

<br/>

[📐 CAD & 3D Hardware Model (Google Drive)](https://drive.google.com/file/d/1Ka_F5BRAXkGf56-jew4U-lu_tEWLgjRq/view) • [📖 Technical Documentation (Markdown)](./DOCUMENTATION.md) • [📄 Technical Documentation (PDF)](./AetherBridge%20—%20Technical%20Documentation%20(Aether_100,%20SIH%202026)%20(1).pdf) • [⚡ Live EOC Operations Console](#-live-eoc-operations-console--ui) • [🛠️ Hardware Circuit](#-hardware-circuit--electrical-design)

---
</div>

## 📌 Executive Summary

During major flood disasters (such as the devastating Barak Valley / Silchar floods), commercial cellular base stations and 5G infrastructure are routinely submerged, overloaded, or knocked offline. First responders in the field rely on standard handheld VHF/UHF walkie-talkies having an RF line-of-sight range of only **3 to 5 km**, while district **Emergency Operations Centres (EOCs)** are situated tens of kilometres away.

Commercial Radio-over-IP (RoIP) appliances are expensive proprietary imports costing ₹2,00,000–₹5,00,000+ per unit and require complete handset replacements.

**SentinelBridge** bridges this critical gap through a unified disaster management platform combining:
1. **AetherBridge (Voice Subsystem)**: A low-cost (**~₹1,200** BOM), plug-and-play Radio-over-IP gateway that connects standard field walkie-talkies to surviving IP links (satellite, mesh, wireline) without modifying field radios or acquiring new RF spectrum.
2. **FloodSense (Predictive AI Subsystem)**: A real-time hydrological forecasting engine and geospatial EOC console utilizing an ensemble of **Bidirectional LSTM** and **Random Forest** models to predict river stage levels and hydrographs hours before peak inundation occurs.

---

## 🔗 Quick Resource Links

| Resource | Description | Direct Link |
| :--- | :--- | :--- |
| 🖼️ **System Poster & Hardware Overview** | High-resolution architectural diagram, leaf node CAD preview & bench prototype setup | [**View System Infographic**](./image.png) |
| 📖 **Technical Documentation (Full Specs)** | Complete Markdown engineering documentation covering circuit schematics, ground-loop math, optocoupler saturation, and RF bench test results | [**Read Documentation**](./DOCUMENTATION.md) |
| 📄 **Hardware Technical Report (PDF)** | Official 7-page technical specification and engineering report document | [**View PDF Document**](./AetherBridge%20—%20Technical%20Documentation%20(Aether_100,%20SIH%202026)%20(1).pdf) |
| 📐 **CAD 3D Enclosure & Enclosure Design** | Complete 3D CAD assembly, STEP/DWG files & mounting chassis for Raspberry Pi + audio interface board | [**Google Drive CAD Link**](https://drive.google.com/file/d/1Ka_F5BRAXkGf56-jew4U-lu_tEWLgjRq/view) |
| 🧠 **ML Model Training Pipeline** | Synthetic & historical hydrograph training script with LSTM and Random Forest export | [`scripts/train_flood_models.py`](./scripts/train_flood_models.py) |
| 💻 **EOC Operations Web Console** | React 19 + TypeScript + Vite + Tailwind CSS interactive tactical console with oscilloscope, live map, and risk engine | [`src/`](./src/) |

---

## 🏛️ System Architecture

```mermaid
flowchart TB
    subgraph FIELD["📡 Field Responder Zone (3-5 km RF Range)"]
        H1["Handheld VHF/UHF Radio\n(Unmodified Standard Handset)"]
    end

    subgraph GATEWAY["⚡ AetherBridge Hardware Gateway (~₹1,200)"]
        direction TB
        K_CONN["Kenwood 2-Pin Connector\n(3.5mm Mic / 2.5mm Spk)"]
        subgraph ISO_BOARD["Galvanic Isolation & PTT Board"]
            GN1["Pi Ground Net"]
            GN2["Radio Ground Net (Isolated)"]
            C_ISO["C3 DC-Blocking Capacitor"]
            OPTO["PC817 Optocoupler"]
            TR_DRIVE["2N3904 NPN PTT Switch (5V Rail)"]
            TRIM["25-Turn Precision Trimmers (RV1, RV2)"]
        end
        RPI["Raspberry Pi 5 Gateway\n(Python Audio Engine, 48kHz Mono)"]
        USB_SC["Low-Noise USB Sound Card"]
        
        K_CONN <--> ISO_BOARD
        ISO_BOARD <--> USB_SC
        USB_SC <--> RPI
        RPI --> TR_DRIVE --> OPTO
    end

    subgraph NETWORK["🌐 Resilient IP Transport"]
        UDP_MESH["Encrypted Peer-to-Peer UDP Stream\n(Bounded Jitter Buffer, 9-17ms RTT)"]
    end

    subgraph EOC["🏢 Emergency Operations Centre (EOC Console)"]
        CONSOLE["SentinelBridge Web Operations Console\n(React 19 + Vite + Web Audio API)"]
        HEADSET["EOC Operator Headset\n(Voice Activity Detector & PTT)"]
        ML_ENGINE["FloodSense AI Engine\n(LSTM + Random Forest Ensemble)"]
        LEAFLET["Barak Basin GIS Telemetry Map"]
    end

    H1 <== "RF Line-of-Sight" ==> K_CONN
    RPI <== "UDP Voice Frames" ==> UDP_MESH
    UDP_MESH <== "Secure Transport" ==> CONSOLE
    CONSOLE <--> HEADSET
    ML_ENGINE --> CONSOLE
    LEAFLET --> CONSOLE
```

---

## 📻 Subsystem 1: AetherBridge Voice Gateway

### 1. The "No New Transmitter" Design Principle
AetherBridge does not contain an RF transmitter. It acts as an intelligent intermediary:
- Audio is tapped directly from the radio's auxiliary/accessory port.
- Push-to-Talk (PTT) is keyed mechanically/electrically using an optocoupler switch.
- Every RF emission originates exclusively from the agency's existing licensed VHF/UHF base station or handheld. No separate wireless spectrum allotment (WPC) is needed.

### 2. The Kenwood 2-Pin Ground-Loop Problem & Resolution
A standard Kenwood two-pin connector combines a 3.5 mm (mic/PTT) and 2.5 mm (speaker) plug. Most off-the-shelf schematics fail because:
- The **2.5 mm sleeve** is the Radio Ground.
- The **3.5 mm sleeve** is the Microphone Return AND PTT sense line.
- When connected to a standard USB sound card with a shared internal ground, the two sleeves are joined, immediately causing a **stuck carrier** that overheats the radio's power amplifier.

**Solution: Strict Two-Ground Net Architecture**
- **Pi Ground Net**: Pi GND, USB sound card sleeves, trimmers RV1/RV2, optocoupler collector.
- **Radio Ground Net**: Radio 2.5 mm sleeve, optocoupler emitter.
- **Isolation**: Nets meet *only* across DC-blocking capacitor $C_3$ (passes AC audio while blocking DC) and the optical gap of the optocoupler. When powered down, resistance between brown and green lines measures **infinite / open circuit**.

```
  [2.5mm Tip] ----( C1 1uF )----+----[ RV1 10k ]----> USB MIC IN (Tip)
                                |
  [2.5mm Sleeve] --( Radio GND )+--( C3 1uF )--------> Pi GND (Sleeve)
  
  [3.5mm Ring] <---( C2 1uF )---+----[ RV2 10k ]<----[ R1 100k ]<--- USB HP OUT
  
  [3.5mm Sleeve] <--( Opto Collector ) 
  [2.5mm Sleeve] <--( Opto Emitter   ) <=== (Driven by 2N3904 from Pi GPIO17)
```

### 3. Audio Chain & PTT Interlock Timers
- **Software Squelch**: Rolling RMS algorithm calculated on 10 ms frames (480 samples @ 48 kHz).
- **PTT Lead-in Time**: Gateway waits **250 ms** after keying PTT before feeding audio, allowing the transmitter power amplifier to stabilize and the remote receiver's squelch gate to open.
- **PTT Tail Time**: PTT is maintained for **350 ms** after speech ends.
- **Fail-Safe Watchdog**: Hard **30-second timeout** automatically releases PTT to prevent stuck transmissions.
- **Half-Duplex Interlock**: Discards microphone input during local transmission to prevent acoustic/electrical oscillation feedback loops.

---

## 🌊 Subsystem 2: FloodSense Predictive AI Engine

FloodSense transforms raw hydro-meteorological sensor streams into actionable disaster mitigation intelligence.

### 1. Machine Learning Hydrological Ensemble
- **Bi-Directional LSTM (Long Short-Term Memory)**: Captures complex temporal dependencies, antecedent soil moisture retention, and cumulative 6-hour/24-hour precipitation lags.
- **Random Forest Regressor**: Estimates non-linear peak river stage thresholds based on basin geometry, upstream telemetry, and sudden cloudburst spikes.
- **Physics-Informed Ensemble**: Blends ML inference with hydrograph mass-conservation bounds to generate **95% Confidence Intervals (CI)**.

```
┌─────────────────────────┐     ┌────────────────────────┐
│ 6h / 24h Rainfall Rates │     │ Antecedent River Stage │
└────────────┬────────────┘     └───────────┬────────────┘
             │                              │
             ▼                              ▼
      ┌──────────────┐              ┌──────────────┐
      │  LSTM Neural │              │ Random Forest│
      │   Network    │              │  Regressor   │
      └──────┬───────┘              └──────┬───────┘
             │                             │
             └──────────────┬──────────────┘
                            ▼
               ┌─────────────────────────┐
               │    Ensemble Blending    │
               │ (Confidence & CI Margin)│
               └────────────┬────────────┘
                            ▼
   ┌─────────────────────────────────────────────────┐
   │ Hydrograph Forecast + CWC 4-Tier Risk Escalation│
   │   [SAFE]  -->  [WATCH]  -->  [WARN]  -->  [CRIT]│
   └─────────────────────────────────────────────────┘
```

### 2. CWC 4-Tier Early Warning Standard
| Alert Tier | River Stage Range | Visual Indicator | Acoustic Telemetry Sound |
| :--- | :--- | :--- | :--- |
| **SAFE** | $< 13.0\text{ m}$ | Emerald Ambient Glow | Standby Chirp (Periodic) |
| **WATCH** | $13.0\text{ m} - 16.0\text{ m}$ | Amber Glow | Single Advisory Ping |
| **WARNING** | $16.0\text{ m} - 19.5\text{ m}$ | High-Intensity Orange Glow | Dual Warning Tone Pulse |
| **CRITICAL** | $\ge 19.5\text{ m}$ (Danger $>19.8\text{ m}$) | Flashing Crimson Strobe | Rapid Evacuation Siren (Continuous with manual silence) |

---

## 📊 Verification & Bench Test Results

As verified during rigorous hardware and RF bench trials (detailed in the [Technical Documentation](./DOCUMENTATION.md) and [PDF Report](./AetherBridge%20—%20Technical%20Documentation%20(Aether_100,%20SIH%202026)%20(1).pdf)):

| Stage | Verification Test | Expected Standard | Measured / Observed Result | Status |
| :---: | :--- | :--- | :--- | :---: |
| **1** | Optocoupler In/Out Ground Isolation | $\infty\ \Omega$ (Open Circuit) | Open Circuit | ✅ **PASS** |
| **2** | Unpowered Board Isolation (Brown to Green) | $\infty\ \Omega$ (Open Circuit) | Open Circuit | ✅ **PASS** |
| **3** | Radio Idle with Gateway Powered (60s) | No RF Carrier Keyed | $0.00\text{ V}$ PTT Trigger / No Transmission | ✅ **PASS** |
| **4** | Software PTT Keying (GPIO17 High) | Reliable TX Closure | **$0.15\text{ V}$** saturation across switch | ✅ **PASS** |
| **5** | Transmit Audio Frequency Response | Clean 1 kHz Modulation | Received clear tone on monitor radio | ✅ **PASS** |
| **6** | Network Round-Trip Time (RTT) | $< 50\text{ ms}$ over WAN | **$9 - 17\text{ ms}$** UDP frame latency | ✅ **PASS** |
| **7** | Network Disconnect Resilience | Graceful Degradation | Radios operate locally; RoIP resumes in $<1\text{s}$ | ✅ **PASS** |

---

## 🗂️ Repository Structure

```tree
SIH_2026/
├── DOCUMENTATION.md                           # Comprehensive Markdown technical documentation
├── AetherBridge — Technical Documentation.pdf  # 7-page engineering report PDF
├── image.png                                  # System architectural infographic & poster
├── image copy.png                             # Core team members and responsibilities banner
├── index.html                                 # HTML5 entry point with viewport config
├── package.json                               # Dependencies (React 19, Vite, Leaflet, Tailwind)
├── tailwind.config.js                         # Design system styling configuration
├── vite.config.ts                             # Vite configuration
│
├── scripts/
│   └── train_flood_models.py                  # Python AI pipeline for flood hydrograph ML
│
└── src/
    ├── App.tsx                                # Main application root & perimeter glow orchestrator
    ├── index.css                              # Tailwind CSS v4 & custom animations
    │
    ├── components/
    │   ├── Header.tsx                         # EOC status bar, risk badge, theme & sound controls
    │   ├── SubsystemsTopBar.tsx               # Quick subsystem switcher & metrics header
    │   ├── SidebarLeft.tsx                    # Telemetry event feed, storm scenario triggers, PTT
    │   ├── SidebarRight.tsx                   # Live station gauges, upstream sensors, logs
    │   ├── BottomStatusBar.tsx                # Gateway link health, latency & system stats
    │   │
    │   ├── AetherBridge/                      # --- RoIP Voice Gateway Components ---
    │   │   ├── AetherBridgePanel.tsx          # Main AetherBridge dashboard layout
    │   │   ├── AetherControls.tsx             # PTT triggers, noise injection, 5G drop/restore
    │   │   ├── AudioOscilloscope.tsx          # Real-time Web Audio API visualizer & RMS squelch
    │   │   ├── CommunicationPipeline.tsx      # Interactive interactive signal flow diagram
    │   │   ├── LatencyBudget.tsx              # Latency breakdown (frame, jitter, PTT lead)
    │   │   ├── LinkHealth.tsx                 # Packet loss, jitter, ping, mesh fallback
    │   │   └── SpectrumChart.tsx              # RF frequency & voice spectrum display
    │   │
    │   └── FloodSense/                        # --- Predictive AI Flood Components ---
    │       ├── FloodSensePanel.tsx            # Main FloodSense dashboard layout
    │       ├── BasinMap.tsx                   # Leaflet interactive GIS basin & station map
    │       ├── FloodChart.tsx                 # Hydrograph charts, predicted peak, 95% CI bands
    │       ├── FloodControls.tsx              # Rainfall/stage sliders & historical scenarios
    │       ├── ModelCards.tsx                 # LSTM vs Random Forest architecture & metrics
    │       └── RiskLadder.tsx                 # CWC 4-stage emergency escalation ladder
    │
    ├── hooks/
    │   └── useSimulationState.ts              # Centralized reactive state & scenario engine
    │
    ├── models/
    │   ├── floodMlEngine.ts                   # In-browser hydrological ML inference engine
    │   └── trainedDataset.json                # Pre-trained weights, baselines, and historical storms
    │
    ├── types/
    │   └── simulation.ts                      # Strict TypeScript interfaces & contracts
    │
    └── utils/
        └── audioSystem.ts                     # Web Audio API synthesizers for acoustic alarms
```

---

## 🚀 Quick Start Guide

### Prerequisites
- **Node.js**: v18.0.0 or higher
- **npm** or **yarn** or **pnpm**
- *(Optional)* **Python 3.10+** (if executing the ML retraining script)

### 1. Installation
```bash
# Clone the repository
git clone https://github.com/techy-geek/Team_Aether.git
cd Team_Aether

# Install dependencies
npm install
```

### 2. Development Server
Start the local Vite development server with Hot Module Replacement (HMR):
```bash
npm run dev
```
Open your browser and navigate to `http://localhost:5173`.

### 3. Production Build
Compile and bundle the production-ready application:
```bash
npm run build
```
Preview the production build locally:
```bash
npm run preview
```

### 4. Running the ML Training Script *(Optional)*
To retrain or benchmark the hydrological models on new rainfall datasets:
```bash
# Install Python ML prerequisites
pip install numpy scikit-learn pandas

# Execute training script
python scripts/train_flood_models.py
```

---

## 🛠️ Hardware Bill of Materials (BOM)

| Component | Part / Spec | Approximate Cost (INR) | Purpose |
| :--- | :--- | :---: | :--- |
| **Optocoupler** | PC817 / 4N35 DIP-4 | ₹15 | Galvanic isolation for PTT trigger |
| **Switching Transistor** | 2N3904 NPN | ₹5 | Pulls optocoupler LED against 5V rail |
| **Precision Trimmers** | 10 kΩ 25-Turn (RV1, RV2) | ₹80 | RX and TX impedance & gain matching |
| **Resistors** | 1 kΩ (Base), 100 kΩ (Attenuator) | ₹5 | Base drive & headphone line attenuation |
| **Capacitors** | 1.0 µF Film / Ceramic (C1, C2, C3) | ₹25 | DC blocking & AC audio passthrough |
| **Audio Interface** | C-Media CM108 USB Sound Card | ₹350 | Dedicated low-noise ADC/DAC |
| **Harness & Plugs** | 2.5 mm + 3.5 mm Kenwood Cable | ₹250 | Interfacing base VHF/UHF radio |
| **Prototyping / Enclosure**| Custom 3D Printed Chassis / PCB | ₹470 | Mechanical housing & mounting |
| **TOTAL (Interface Subsystem)** | — | **~₹1,200** | *(Excludes Raspberry Pi & Radio)* |

---

## 💡 Team & Hackathon Credits

[![Team Aether_100 Members & Leads](./image%20copy.png)](./image%20copy.png)

### 👥 Team Members & Roles

| Member Name | Role & Specialization | Subsystem & Key Responsibilities |
| :--- | :--- | :--- |
| 👑 **Jay Vishwakarma** *(Leader)* | **Hardware Lead** | Overall project lead, galvanic isolation board design, 2N3904 + PC817 PTT switching circuit & hardware bench validation |
| 📡 **Ayush Thayyil** | **Communication Lead** | Radio-over-IP (RoIP) audio pipeline, 48 kHz mono frame engine, UDP streaming & network resilience protocols |
| 📊 **Ayushman Swain** | **PPT Lead** | Project presentation decks, technical diagrams, architectural flowcharts & documentation assets |
| 🔬 **Nishita Sarma** | **R&D Lead** | Disaster communication literature survey, hydrological telemetry research & CWC 4-tier protocol integration |
| 🧠 **Anurag Sharma** | **AI/ML Lead** | FloodSense Bi-LSTM & Random Forest hydrological ML engine, model training pipelines & real-time stage inference |
| 📐 **Ankit Yadav** | **CAD Modelling Lead** | 3D mechanical enclosure modeling, Leaf Node CAD preview, Raspberry Pi + PCB mounting chassis & STEP/DWG exports |

---

### 🏛️ Institution & Mentorship
* **Event**: Smart India Hackathon (SIH) 2026
* **Problem Statement**: `SIH26223` (Disaster Management / Hardware)
* **Team**: **Team Aether_100**
* **Team ID**: `NITS_100`
* **Institution**: National Institute of Technology Silchar (NIT Silchar)
* **Project Mentor**: Dr. Koushik Guha

---

## 📜 License & Open Source

This project is released under the **MIT License**. Open-source contribution and civic disaster deployments are encouraged.
