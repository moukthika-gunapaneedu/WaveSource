# WaveSource

WaveSource is a Python-based pipeline designed to **automatically extract, validate, and structure metadata from historical tsunami marigram (tide gauge) images**. These marigrams are typically stored as high-resolution TIFF scans and contain critical information such as station location, event date, scale, and region identifiers that are not machine-readable by default.

The project combines **OCR, rule-based parsing, official metadata allow-lists, and a human-in-the-loop review step** to convert unstructured archival images into a structured dataset suitable for scientific analysis and archival integration.

---

## Project Background

NOAA's National Centers for Environmental Information (NCEI) maintains one of the world's largest archives of tsunami marigram records. Most of these records were digitized from microfilm under the Climate Data Modernization Program (CDMP) and are stored as scanned TIFF images.

While a collection-level inventory exists (folders, scan counts, date ranges), **item-level metadata lives inside the images themselves** as printed or handwritten headers. Extracting this metadata manually is time-consuming and error-prone.

WaveSource was built to bridge this gap by automating most of the metadata extraction process while preserving accuracy through controlled validation and manual review where needed.

---

## What the Pipeline Does

1\. Reads marigram image files stored in Google Drive folders (recursively)

2\. Runs OCR (Tesseract) with multiple preprocessing strategies to maximize text quality

3\. Parses key metadata fields using regex-based and lightweight structural heuristics

4\. Validates extracted values against **official NOAA descriptor allow-lists**

5\. Resolves station codes using the **IOC Sea Level Monitoring Facility station list**

6\. Optionally geocodes latitude and longitude from validated place names

7\. Writes structured, standardized rows into an Excel workbook

8\. Supports a **human-in-the-loop review** step for ambiguous or missing fields

---

## Output Schema

Each processed marigram produces a single row with the following fields:

| Column Name | Description |

|------------|-------------|

| FILE_NAME | Drive-relative image path (including folder structure) |

| COUNTRY | NCEI country name |

| STATE | NCEI state or prefecture name |

| LOCATION | NCEI location name |

| LOCATION_SHORT | IOC Sea Level Monitoring station code (strict match only) |

| REGION_CODE | NCEI tsunami region code (2-digit, explicit only) |

| START_RECORD | Reserved column (not currently parsed) |

| END_RECORD | Reserved column (not currently parsed) |

| TSEVENT_ID | Reserved column (not currently parsed) |

| TSRUNUP_ID | Reserved column (not currently parsed) |

| RECORDED_DATE | Date of tsunami event (`YYYY/MM/DD`) |

| LATITUDE | Decimal latitude (optional geocoding) |

| LONGITUDE | Decimal longitude (optional geocoding) |

| IMAGES | Number of images per record (currently set to `1` per file) |

| SCALE | Scale factor normalized to `1:NN` |

| MICROFILM_NAME | Microfilm identifier (set manually or from folder name) |

| COMMENTS | OCR confidence, processing notes, or anomalies |

> Columns marked as *reserved* are included for schema completeness and future extension.

---

## Authoritative Metadata Sources

WaveSource strictly relies on **official reference lists** and never invents metadata values.

### NOAA NCEI Descriptor APIs

- Countries  

  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/countries

- States  

  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/states

- Locations (paginated)  

  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/locations

- Tsunami Region Codes  

  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/regions

### IOC Sea Level Monitoring Facility

- Station codes (`LOCATION_SHORT`)  

  https://www.ioc-sealevelmonitoring.org/list.php

---

## Automation vs Manual Review

WaveSource is designed as a **human-in-the-loop system**:

- If a value **matches an official allow-list**, it is accepted automatically

- If a value **does not match**, the OCR result is preserved and flagged

- Region codes and IOC station codes are populated **only if explicitly present and valid**

- Ambiguous or missing fields are reviewed and corrected manually

This approach ensures high accuracy while dramatically reducing manual data entry time without introducing speculative metadata.

---

## Requirements

### Python Dependencies

```bash

pip install -r requirements.txt

```

### System Dependencies

#### Tesseract OCR

-   **Ubuntu**

    `sudo apt-get install tesseract-ocr`

-   **macOS**

    `brew install tesseract`

-   **Windows**\
    Install Tesseract and add it to your system `PATH`.

Verify installation:

`tesseract --version`

* * * * *

Google Drive Access
-------------------

The pipeline reads images directly from **Google Drive**.

### Setup

1.  Create a Google Cloud project

2.  Enable the **Google Drive API**

3.  Create OAuth credentials (**Desktop application**)

4.  Download the credentials file as `credentials.json`

5.  Place `credentials.json` in the same directory as the script

On first run, a browser window opens to authorize access and creates a local `token.json`.

* * * * *

Running the Pipeline
--------------------

By default, the pipeline runs in **non-interactive batch mode**.\
The `--interactive` flag is intended for cleanup or validation passes only.

`python drive_marigram_hitl_to_excel.py\
  --folder-ids <FOLDER_ID_1> <FOLDER_ID_2>\
  --out-xlsx ./Tsunami_Microfilm_Inventory_Output.xlsx\
  --out-csv ./Tsunami_Microfilm_Inventory_Output.csv\
  --cache-dir ./_drive_cache\
  --save-ocr ./_ocr_audit\
  --resume\
  --microfilm-name-from-folder`

* * * * *

Key Flags
---------

-   `--interactive`\
    Enables human-in-the-loop review prompts

-   `--resume`\
    Skips files already marked `"status":"ok"` in the progress log

-   `--enable-geocode`\
    Enables latitude/longitude geocoding (rate-limited)

-   `--microfilm-name-from-folder`\
    Sets `MICROFILM_NAME` from the top-level Drive folder name

* * * * *

Outputs
-------

-   Excel workbook containing structured marigram metadata

-   Optional CSV mirror output for review and diffing

-   Optional OCR audit artifacts:

    -   Raw OCR text files

    -   Best preprocessed image per marigram

-   Progress log (`.jsonl`) for resumable processing

* * * * *

Notes & Limitations
-------------------

-   OCR quality varies depending on scan clarity and handwriting

-   Geocoding relies on external services and may be rate-limited

-   Some historical headers require manual interpretation

-   The pipeline intentionally avoids heuristic guessing to preserve archival integrity

* * * * *

License
-------

This project is intended for research, archival processing, and educational use.\
Please review NOAA and IOC data usage guidelines before redistributing derived datasets.

* * * * *

Acknowledgements
----------------

-   NOAA National Centers for Environmental Information (NCEI)

-   IOC Sea Level Monitoring Facility

-   Climate Data Modernization Program (CDMP)
