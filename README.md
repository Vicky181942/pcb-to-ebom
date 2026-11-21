# PCB → eBoM Generator

Automatically turn a photo or optical scan of a populated PCB into a consolidated **Electronic Bill of Materials (eBoM)**. The app detects components with a computer-vision pipeline, optionally refines identification with a multimodal vision LLM, aggregates the results by Manufacturer Part Number (MPN), footprint and value, and exports a procurement-ready BoM.

Built with Python and Streamlit.

---

## Features

- **Two input modes** – choose a board from the built-in *Sample Board Gallery* (e.g. Arduino Uno Rev3) or *upload your own optical scan / photo*.
- **Computer Vision Engine**
  - Optional **CLAHE contrast enhancement** for uneven lighting
  - Adjustable **confidence threshold**, **NMS IoU threshold** and **minimum component area (px)**
- **Multimodal Vision LLM** support for reading markings and refining component classification.
- **Visual Inspection Studio**
  - Colour-coded AI detection overlay with RefDes + part labels
  - Original image view and side-by-side comparison
  - **Component Crop Inspector** – zoom into any detected part (e.g. `U1 – IC (ATMEGA328P-AU)`) and view specs & sourcing info
- **Automated eBoM Table** – components aggregated by category, MPN, description, package, value, quantity and reference designators, with a text filter (e.g. `U1`, `Resistor`, `10k`).
- **Procurement & Class Analytics** – charts summarising component classes and quantities.
- **SMD Code Decoder Utility** – decode SMD marking codes (e.g. resistor `103` → 10 kΩ, capacitor codes).
- **Export** the eBoM as **Excel (.xlsx)**, **CSV (.csv)** or **JSON (IPC schema)**.

---

## Screenshots


<img width="1900" height="902" alt="pcb2board" src="https://github.com/user-attachments/assets/8f4e85f7-c7bc-4b6f-85ba-b5df3a6fcef8" />
 <img width="1902" height="902" alt="table" src="https://github.com/user-attachments/assets/e26a0304-4aac-4229-9fcf-b7ae3056a08e" />
 


## How It Works

```
 PCB image ──► Pre-processing (CLAHE) ──► Component detection (CV model)
                                              │
                                    NMS + confidence + area filters
                                              │
                                  Vision LLM refinement (optional)
                                              │
                         Ref-Des assignment & MPN / footprint mapping
                                              │
                          Aggregation by MPN + package + value
                                              │
                         eBoM table  ──►  XLSX / CSV / JSON export
```

---

## Example eBoM Output

| Item # | Category  | MPN             | Description                         | Package        | Value | Qty | Ref Des   |
|-------:|-----------|-----------------|-------------------------------------|----------------|-------|----:|-----------|
| 1 | Capacitor | CC0805KRX7R9BB104 | MLCC 0805 SMD 50V X7R              | 0805 SMD       | 100 nF | 6  | C1, C2, C3, … |
| 3 | Connector | PIN-HEADER-2.54   | Standard pin header                | 2.54mm Header  | –      | 3  | J1, J2, J3 |
| 4 | Diode     | 1N4148W           | SMD switching diode 20mA           | SOD-123        | –      | 16 | D1, D2, D3, … |
| 5 | IC        | ATMEGA328P-AU     | 8-bit AVR MCU, 32KB Flash, 20MHz   | QFP / BGA / DIP| –      | 1  | U1 |
| 9 | Resistor  | RC0805JR-10K      | Thick film chip resistor ±1% 1/8W  | 0805 SMD       | 10 kΩ  | 13 | R1, R2, R3, … |
| 10| Transistor| MMBT3904          | NPN small signal 40V 200mA         | SOT-23         | –      | 5  | Q1, Q2, Q3, … |

---

## Installation

### Prerequisites
- Python 3.9+
- (Optional) API key for the Vision LLM provider you use

### Setup

```bash
git clone https://github.com/<your-username>/pcb-to-ebom.git
cd pcb-to-ebom

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### Configuration (optional)

Create a `.env` file for the multimodal LLM:

```env
LLM_API_KEY=your_api_key_here
```

---

## Usage

```bash
streamlit run app.py
```

Then open <http://localhost:8501>.

1. **Input Image** – pick a sample PCB or upload a scan/photo.
2. **Computer Vision Engine** – tune CLAHE, confidence, NMS IoU and minimum area.
3. **Multimodal Vision LLM** – enable and configure if desired.
4. Review results in the tabs:
   - *Visual Inspection Studio*
   - *Automated eBoM Table*
   - *Procurement & Class Analytics*
   - *SMD Code Decoder Utility*
5. Download the eBoM as **Excel**, **CSV** or **JSON**.

### Tuning Tips

| Parameter | Lower value | Higher value |
|---|---|---|
| Confidence threshold | More detections, more false positives | Fewer, more reliable detections |
| NMS IoU threshold | Removes more overlapping boxes | Keeps more overlapping boxes |
| Min component area | Detects tiny parts (0402, etc.) | Ignores small noise/artifacts |

For best results use a top-down, evenly lit, high-resolution image and enable CLAHE for low-contrast boards.

---

## Project Structure

> Adjust to match your repository.

```
pcb-to-ebom/
├── app.py                 # Streamlit entry point
├── requirements.txt
├── .env.example
├── core/
│   ├── detector.py        # CV detection, NMS, filtering
│   ├── preprocess.py      # CLAHE and image utilities
│   ├── llm_refine.py      # Vision LLM identification
│   ├── ebom.py            # Aggregation & BoM generation
│   └── smd_decoder.py     # SMD marking decoder
├── assets/
│   └── samples/           # Sample board gallery
├── docs/                  # Screenshots
└── README.md
```

---

## Export Formats

- **Excel (.xlsx)** – formatted table for sharing with purchasing/manufacturing.
- **CSV (.csv)** – plain tabular data for import into other tools.
- **JSON (IPC schema)** – structured output aligned with IPC-style BoM exchange for tooling integration.

---

## Limitations

- Part numbers inferred visually are **candidates** – verify MPNs against the schematic or datasheets before ordering.
- Values for passives (resistors, capacitors) depend on legible markings.
- Heavily occluded, low-resolution or angled images reduce accuracy.
- Package ambiguity is possible for ICs (e.g. QFP vs. QFN) without a clear marking.



