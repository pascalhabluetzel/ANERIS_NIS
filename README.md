# Marine Invasiveness Analysis Script  
*A tool for combining OBIS, Marine Regions, and WRiMS data to assess marine species invasiveness across locations.*

---

## 📌 Overview

This repository provides a Python script that automates the analysis of **marine species occurrences**, integrating data from:

- **OBIS** (Ocean Biodiversity Information System)  
- **Marine Regions** gazetteer  
- **WoRMS / WRiMS** (World Register of Marine Species / World Register of Introduced Marine Species)  

For every **location + species** pair, the script:

1. Converts geographic coordinates (DMS → decimal degrees)  
2. Retrieves OBIS occurrences and calculates **sea-route distance** to the nearest known record  
3. Identifies Marine Regions containing the location  
4. Determines WRiMS invasiveness status (native, introduced, invasive, or unrecorded)  
5. Writes all results into a **single TSV output file**

This workflow enables large-scale biogeographic screening, invasion monitoring, and rapid detection of potential non-native occurrences.

---

## ✨ Features

- ✔️ Batch analysis of many species × location combinations  
- ✔️ Accurate **over-water distance** calculations using searoute  
- ✔️ Integration of multiple authoritative marine datasets  
- ✔️ Clean TSV output suitable for R, Python, GIS, or spreadsheets  
- ✔️ Human-readable invasiveness labels  
- ✔️ Caches repeated API queries for speed  

---

## 📂 Input Files

The script expects two **tab-separated-value (TSV)** files:

---

### 1. `locations.tsv`

| Column | Description |
|--------|-------------|
| `locationID` | Unique identifier for the location |
| `verbatimLatitude` | Latitude in DMS format (e.g. `51°06'39.3"N`) |
| `verbatimLongitude` | Longitude in DMS format (e.g. `2°36'13.6"E`) |

**Example:**

```tsv
locationID	verbatimLatitude	verbatimLongitude
LOC001	51°06'39.3"N	2°36'13.6"E
LOC002	50°58'10.2"N	1°35'42.1"W
```

---

### 2. `species.tsv`

| Column | Description |
|--------|-------------|
| `locationID` | Must match the IDs in `locations.tsv` |
| `scientificName` | Species name (OBIS/WoRMS compatible) |
| `taxonRank` | Optional |

**Example:**

```tsv
locationID	scientificName	taxonRank
LOC001	Carcinus maenas	species
LOC001	Crassostrea gigas	species
LOC002	Carcinus maenas	species
```

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 2. Create and activate a Python environment (optional)

```bash
python -m venv env
source env/bin/activate      # macOS / Linux
# or:
env\Scripts\activate         # Windows
```

### 3. Install dependencies

```bash
pip install pandas tqdm requests searoute
```

---

## ▶️ Usage

### 1. Set file paths in the script

Modify the following section near the top:

```python
LOCATIONS_FILE = "locations.tsv"
SPECIES_FILE   = "species.tsv"
OUTPUT_FILE    = "analysis_results.tsv"
```

### 2. Run the script

```bash
python batch_invasion_analysis.py
```

Progress bars will appear for:

- OBIS sea-route distance calculations  
- WRiMS status computations  

Upon completion, the script prints:

```
Done. Results written to: analysis_results.tsv
```

---

## 🧪 Output: `analysis_results.tsv`

Each row represents a **species occurring at a given location**.

### Key columns

| Column | Meaning |
|--------|---------|
| `locationID` | Location identifier |
| `decimalLatitude`, `decimalLongitude` | Computed decimal coordinates |
| `scientificName` | Species name |
| `closest_obis_lat`, `closest_obis_lon` | Nearest OBIS occurrence |
| `distance_over_water_km` | Sea-route distance to nearest OBIS record |
| `wrims_status_new_location` | WRiMS-based label at your location |
| `wrims_status_closest_obis` | WRiMS-based label at the closest OBIS location |

### WRiMS labels include:

- **probably invasive**  
- **probably introduced**  
- **probably native**  
- **no WRiMS distribution record**

---

## 🧠 Interpreting Results

### Sea-route distance

- **Small (<100 km):** likely within known range  
- **Large (> several hundred km):** possible new introduction, range expansion, or sampling gap  

### WRiMS invasiveness categories

| Label | Interpretation |
|-------|----------------|
| **probably invasive** | Strong evidence of invasiveness in the region |
| **probably introduced** | Non-native but not clearly invasive |
| **probably native** | Distribution present without alien status |
| **no WRiMS distribution record** | No WRiMS information available |

Comparing the status at the observation site and the nearest OBIS occurrence can reveal biogeographic transitions or early-stage invasions.

---

## ⚠️ Caveats

- Requires **internet access** for OBIS, Marine Regions, and WoRMS APIs  
- Long-running for large datasets  
- OBIS coverage varies by taxon  
- WRiMS does not document every alien or invasive species  
- The script provides **screening-level insights**, not definitive ecological conclusions  

