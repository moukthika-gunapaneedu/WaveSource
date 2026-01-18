# WaveSource: Tsunami-Marigram-Metadata-Extraction

This is a Python-based tool designed to automatically extract, parse, and structure metadata from historical tsunami marigram records. These marigrams are often stored as TIFF images and contain critical tide gauge information such as latitude, longitude, event date, and comments. This project aims to make these records more discoverable, structured, and ready for further scientific analysis.

## Project Background
NOAA’s National Centers for Environmental Information (NCEI) maintains one of the largest archives of historical tsunami marigram (tide gauge) records. These records span from the mid-1800s to the late 20th century, capturing worldwide tsunami events across thousands of coastal stations.  

Most marigrams exist as high-resolution TIFF scans from microfilm rolls, digitized under NOAA’s Climate Data Modernization Program (CDMP). However, these scans often lack structured metadata, making it difficult for researchers to search, analyze, and integrate them into modern databases.  

This project was created to **bridge that gap** by:
- Running OCR to extract text from the marigram images.
- Parsing geographic and temporal metadata into standardized formats.
- Organizing the results into a structured CSV/Excel dataset.
- Enabling researchers to query, validate, and extend the historical tsunami record.


## Features
-  Extracts text from marigram TIFF images using OCR (Tesseract).
-  Cleans, normalizes, and parses metadata into structured formats.
-  Handles multiple latitude/longitude formats (decimal, signed, DMS).
-  Detects and standardizes event dates from handwritten or printed marigrams.
-  Outputs metadata into CSV/Excel for downstream research.
-  Includes regex-based parsing patterns for robust extraction.
-  Designed for extensibility to accommodate additional metadata fields.

## How It Works
1. Input raw TIFF marigram scans.
2. Run OCR (Tesseract) to extract text from images.
3. Apply regex-based patterns to detect latitude, longitude, event dates, and comments.
4. Normalize values into consistent formats (decimal degrees, ISO 8601 dates).
5. Save results into a structured dataset (CSV/Excel).

## Installation
```bash
git clone https://github.com/<your-username>/Tsunami-Marigram-Metadata-Extractor.git
cd Tsunami-Marigram-Metadata-Extractor
pip install -r requirements.txt

# Usage
python extract_metadata.py --input ./data/marigrams/ --output ./output/metadata.csv
```

## Output Columns

```bash
FILE_NAME, COUNTRY, STATE, LOCATION, LOCATION_SHORT, REGION_CODE,
START_RECORD, END_RECORD, TSEVENT_ID, TSRUNUP_ID, RECORDED_DATE,
LATITUDE, LONGITUDE, IMAGES, SCALE, MICROFILM_NAME, COMMENTS
```

## Regex Patterns
The extractor uses multiple regex strategies to capture different latitude/longitude formats:

- Decimal with N/S/E/W → 19.73 N 155.08 W

- Signed decimal only → 19.73, -155.08

- DMS format → 19°43'N 155°05'W

## Tips & Troubleshooting

- Tesseract not found: ensure it’s installed and on PATH (see installation notes).

- OCR misses/streaking: the script already tries several binarizations; you can add deskew or morphology in the preprocessing function if needed.

- Geocoding ambiguous: add the place to locations.txt, or specify more context in the image header (COUNTRY/STATE/LOCATION). Nominatim is rate-limited; this script uses ~1 req/sec.

- Region code empty: ensure a valid IOC code (e.g., [85]) is present in the header or provide an explicit CSV mapping.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request to suggest improvements.




