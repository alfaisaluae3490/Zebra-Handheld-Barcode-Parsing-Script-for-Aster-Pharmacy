# Zebra Handheld Barcode Parsing Script to Parse Specific GS1 Barcodes

This repository contains a custom script developed for Zebra handheld devices used at Aster Pharmacy. The script is designed to parse specific GS1 barcodes, extracting the Global Trade Item Number (GTIN) and Serial Number, and reformatting the output for seamless integration with internal systems.


## Project Overview

### Why This Script Was Developed

In a fast-paced pharmacy environment like Aster, efficiency and accuracy in data capture are critical. The standard output from scanning GS1-128 barcodes on medication packaging was not directly compatible with our inventory management software. The raw barcode data contained multiple application identifiers (AIs) and variable-length data fields, which required manual editing or complex backend processing.

This script was developed to automate the parsing process directly on the Zebra handheld device. It intelligently identifies and extracts the essential information, the product's GTIN (AI `01`) and its unique Serial Number (AI `21`), and reformats it into a clean, usable string before it is sent to the host application. This eliminates errors, speeds up the receiving and dispensing workflow, and reduces the data processing load on our central systems.

### What It Does

The script intercepts the data from a scanned barcode and performs the following actions:

1.  **Checks for a GTIN:** It first verifies if the barcode starts with the Application Identifier `01` for the GTIN.
2.  **Extracts GTIN and Serial Number:** If a GTIN is present, it isolates the 14-digit GTIN and then searches for the Serial Number, which is identified by the AI `21`.
3.  **Handles Data Separators:** It correctly processes the invisible Group Separator character (`<GS>`) that often terminates a variable-length field like the serial number.
4.  **Reconstructs the Data:** The script then rebuilds the scan data to include only the GTIN and the Serial Number, each preceded by their respective AIs.
5.  **Adds a Suffix:** Finally, it appends a Carriage Return (`<CR>`) character to the end of the processed data. This acts as an "Enter" key, automatically submitting the cleaned data to the application, saving the user an extra step.

If the barcode does not conform to the expected `01` and `21` structure, the script simply passes the original data through with a Carriage Return suffix, ensuring that other types of barcodes still work as expected.

## How It Works

The script is triggered every time a barcode is successfully scanned (`_event == "decode"`). It then inspects the scanned data (`_data`) and applies a series of logical steps to parse it.

Here is a simplified breakdown of the script's logic:

IF the scanned data begins with "01":
Extract the 14-digit GTIN.
Look for the "21" identifier for the Serial Number.
IF "21" is found:
Extract the Serial Number.
Clean up the Serial Number by removing any trailing characters.
Rebuild the data as: "01" + GTIN + "21" + Serial Number.
ELSE (if "21" is not found):
Keep the original data. ELSE (if the data does not begin with "01"):
Keep the original data.
FINALLY:
Append a Carriage Return to the data to auto-submit it.
Plain Text

This ensures that only the relevant information is passed to the foreground application, in the correct format.

## The Script

This script is written for Zebra's DataWedge software, which uses a JavaScript-like syntax for its scripting engine.

```javascript
// This script is triggered on every successful barcode scan.
if (_event == "decode") {

    // Check if the barcode data starts with the Application Identifier (AI) for GTIN ("01").
    if (sub(_data, "0", "2") == "01") {

        // Extract the 14-digit GTIN, which follows the "01" AI.
        gtin = sub(_data, "2", "16");

        // Get the remaining part of the barcode string after the GTIN.
        after_gtin = sub(_data, "16");

        // Find the position of the AI for Serial Number ("21").
        pos_21 = find(after_gtin, "21");

        // If the "21" AI is found...
        if (pos_21 != "") {

            // Get the substring that starts from "21".
            after_21 = sub(after_gtin, pos_21);
            // The serial number is the part after the "21" AI.
            serial = sub(after_21, "2");

            // Check for a Group Separator character (\x1D), which marks the end of a variable-length field.
            gs_pos = find(serial, "\x1D");
            if (gs_pos != "") {
                // If found, truncate the serial number to remove the separator and anything after it.
                serial = sub(serial, "0", gs_pos);
            }

            // Reconstruct the data string with only the GTIN and Serial Number, and add a Carriage Return.
            _data = concat("01", gtin, "21", serial, "\x0D");

        } else {
            // If "21" is not found, just append a Carriage Return to the original data.
            _data = concat(_data, "\x0D");
        }

    } else {
        // If the barcode does not start with "01", append a Carriage Return to the original data.
        _data = concat(_data, "\x0D");
    }
}

