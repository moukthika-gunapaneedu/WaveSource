# WaveSource

WaveSource is a Python-based pipeline designed to **extract, validate, and structure metadata from historical tsunami marigram (tide gauge) images**. These marigrams are typically stored as high-resolution TIFF scans and contain critical information such as station location, dates, scale, and region identifiers that are not machine-readable by default.

The pipeline combines **OCR, rule-based parsing, official metadata allow-lists, and a human-in-the-loop review step** to convert unstructured archival images into a structured dataset suitable for scientific analysis and archival integration.



## Project Background

NOAA’s National Centers for Environmental Information (NCEI) maintains one of the world’s largest archives of tsunami marigram records. Most records were digitized from microfilm under the Climate Data Modernization Program (CDMP) and stored as scanned TIFF images.

While a collection-level inventory exists (folders, scan counts, date ranges), **item-level metadata lives inside the images themselves** as printed or handwritten headers. Extracting this metadata manually is time-consuming and error-prone.

WaveSource bridges this gap by automating most of the metadata extraction process while preserving accuracy through controlled validation and manual review where needed.



## What the Pipeline Does

1. Reads marigram image files stored in Google Drive folders (recursively)
2. Runs OCR (Tesseract) with multiple preprocessing strategies to improve text extraction
3. Parses key metadata fields using regex-based and lightweight structural heuristics
4. Validates extracted values against **official NOAA descriptor allow-lists** 
5. Resolves station codes using the **IOC Sea Level Monitoring Facility station list** (strict match only)
6. Optionally geocodes latitude and longitude from place names (rate-limited)
7. Appends structured rows into an Excel workbook
8. Supports a **human-in-the-loop review** mode for missing/ambiguous fields
9. Writes a progress log so runs can be resumed safely



## Output Schema

Each processed marigram produces a single row with the following fields:

| Column Name      | Description |
|------------------|-------------|
| FILE_NAME        | Drive-relative image path (including folder structure) |
| COUNTRY          | NOAA/NCEI country descriptor (validated against NOAA allow-list) |
| STATE            | NOAA/NCEI state/prefecture descriptor (validated against NOAA allow-list) |
| LOCATION         | NOAA/NCEI location descriptor (validated against NOAA allow-list) |
| LOCATION_SHORT   | IOC Sea Level Monitoring station code (strict country+location match only) |
| REGION_CODE      | NOAA/NCEI tsunami region code (2-digit, explicit only) |
| START_RECORD     | Reserved column (schema compatibility / future extension) |
| END_RECORD       | Reserved column (schema compatibility / future extension) |
| TSEVENT_ID       | Reserved column (schema compatibility / future extension) |
| TSRUNUP_ID       | Reserved column (schema compatibility / future extension) |
| RECORDED_DATE    | Extracted date normalized to `YYYY/MM/DD` (when present) |
| LATITUDE         | Decimal latitude (optional geocoding) |
| LONGITUDE        | Decimal longitude (optional geocoding) |
| IMAGES           | Number of images per record (currently set to `1` per file) |
| SCALE            | Scale factor normalized to `1:NN` (when present) |
| MICROFILM_NAME   | Microfilm identifier (set manually or from folder name) |
| COMMENTS         | OCR confidence + processing notes (psm/oem, anchors, path) |



## Authoritative Metadata Sources

WaveSource relies on official reference lists and intentionally avoids inventing values.

### NOAA NCEI Descriptor APIs
- Countries:  
  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/countries
- States:  
  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/states
- Locations (paginated):  
  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/locations
- Tsunami Region Codes:  
  https://www.ngdc.noaa.gov/hazel/hazard-service/api/v1/descriptors/tsunamis/marigrams/regions

### IOC Sea Level Monitoring Facility
- Station codes (`LOCATION_SHORT`):  
  https://www.ioc-sealevelmonitoring.org/list.php


## Automation vs Manual Review

WaveSource is designed as a **human-in-the-loop system**:

- If a value **matches an official allow-list**, it is accepted automatically
- If a value **does not match**, the OCR result is preserved and flagged for review
- Region codes and IOC station codes are populated **only if explicitly present and valid**
- Manual review (`--interactive`) is intended for cleanup/validation passes

This approach improves throughput while preserving archival integrity.



## Requirements

### Python Dependencies
```bash
pip install -r requirements.txt
```

### System Dependency: Tesseract OCR

-   Ubuntu:

    `sudo apt-get install tesseract-ocr`

-   macOS:

    `brew install tesseract`

-   Windows: install Tesseract and add it to your system `PATH`

Verify:

`tesseract --version`


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

> Do not commit `credentials.json` or `token.json` to GitHub.


Running the Pipeline
--------------------

### Non-interactive (recommended for batch runs)

```bash
python Tsunami_Marigram.py \
  --folder-ids <FOLDER_ID_1> <FOLDER_ID_2> \
  --out-xlsx ./Tsunami_Microfilm_Inventory_Output.xlsx \
  --cache-dir ./_drive_cache \
  --save-ocr ./_ocr_audit \
  --resume \
  --microfilm-name-from-folder
```

### Interactive review mode (for cleanup passes)
```bash
python Tsunami_Marigram.py\
  --folder-ids <FOLDER_ID_1>\
  --out-xlsx ./Tsunami_Microfilm_Inventory_Output.xlsx\
  --interactive\
  --resume
```

### Optional: enable geocoding (rate-limited)

```bash
python Tsunami_Marigram.py\
  --folder-ids <FOLDER_ID_1>\
  --out-xlsx ./Tsunami_Microfilm_Inventory_Output.xlsx\
  --enable-geocode\
  --resume
```

> **Note (Windows PowerShell):**  
> PowerShell does not support line continuation with `\`.  
> Run the command on a single line instead.

Key Flags
---------

-   `--interactive`\
    Enables human-in-the-loop review prompts

-   `--resume`\
    Skips files already marked `"status":"ok"` in the progress log (`.jsonl`)

-   `--enable-geocode`\
    Enables latitude/longitude geocoding (rate-limited)

-   `--microfilm-name-from-folder`\
    Sets `MICROFILM_NAME` from the top-level Drive folder name

-   `--save-ocr <dir>`\
    Saves OCR text + best preprocessed image per marigram for audit/debugging
    
Outputs
-------

-   Excel workbook (`.xlsx`) containing structured marigram metadata

-   Optional OCR audit artifacts (if `--save-ocr` is used):

    -   Raw OCR text files (`.txt`)

    -   Best preprocessed image per marigram (`*_best.png`)

-   Progress log (`.jsonl`) for resumable processing

Notes & Limitations
-------------------

-   OCR quality varies depending on scan clarity and handwriting

-   IOC station codes require an **exact (COUNTRY, LOCATION)** match; otherwise left blank

-   Geocoding relies on external services and may be rate-limited

-   The pipeline intentionally avoids heuristic guessing to preserve archival integrity

License
-------

Intended for research, archival processing, and educational use.\
Please review NOAA and IOC data usage guidelines before redistributing derived datasets.


Acknowledgements
----------------

-   NOAA National Centers for Environmental Information (NCEI)

-   IOC Sea Level Monitoring Facility

-   Climate Data Modernization Program (CDMP)
