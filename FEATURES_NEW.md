# Yarnl - New Features Documentation

This document describes the new features added to Yarnl to expand pattern management and inventory capabilities.

## Table of Contents
1. [Unique Pattern IDs (Inventory Numbering)](#unique-pattern-ids)
2. [Image Upload Support](#image-upload-support)
3. [OCR Integration](#ocr-integration)
4. [Thread & Material Inventory](#thread--material-inventory)
5. [Expanded Pattern Metadata](#expanded-pattern-metadata)
6. [Barcode Support](#barcode-support)
7. [Threadloop Integration](#threadloop-integration)

---

## Unique Pattern IDs

### Overview
Every pattern now receives a unique inventory ID automatically upon creation. This allows you to maintain your own numbering system for archival and organization purposes.

### Features
- **Automatic Generation**: Each pattern gets a unique ID in format `PAT-{TIMESTAMP}-{RANDOM}`
- **No Duplicates**: The system ensures no two patterns share the same ID
- **Searchable**: Can search patterns by inventory ID in the library

### API
- **Attribute**: `inventory_id` (VARCHAR)
- **Generated**: Automatically on pattern creation
- **Example**: `PAT-1A2B3C-F4G5H6JK`

### Usage
When uploading patterns, the inventory ID will be automatically assigned and displayed in the pattern details.

---

## Image Upload Support

### Overview
Yarnl now supports uploading image files in addition to PDFs. Supported formats:
- **PDF** (original support)
- **JPEG** (.jpg, .jpeg)
- **PNG** (.png)
- **WebP** (.webp)
- **TIFF** (.tiff)

### Features
- **Automatic Thumbnails**: Images are automatically converted to thumbnails for library view
- **File Type Detection**: System automatically detects file type
- **Large File Support**: Increased upload limit to 100MB (from 50MB)
- **Metadata Extraction**: Can extract text from images via OCR

### API Changes
- **Endpoint**: `POST /api/patterns` (unchanged)
- **New Field**: `file_type` - automatically set to 'pdf', 'image', or 'markdown'
- **Thumbnail Generation**: Automatically creates 200x250px thumbnails for images

### Frontend Integration
Update the file upload form to accept image files:

```html
<input type="file" accept=".pdf,.jpg,.jpeg,.png,.webp,.tiff" />
```

### Example Upload
```javascript
const formData = new FormData();
formData.append('pdf', imageFile); // Note: field is still named 'pdf' for compatibility
formData.append('name', 'My Pattern');
formData.append('category', 'Amigurumi');

fetch('/api/patterns', {
  method: 'POST',
  body: formData
});
```

---

## OCR Integration

### Overview
Optical Character Recognition (OCR) allows automatic text extraction from images and PDFs. This makes pattern content searchable and enables text-based notes.

### Features
- **Local Processing**: Uses Tesseract.js for client-side OCR capability
- **Automatic Processing**: Option to enable OCR during upload
- **Post-Processing**: Can run OCR on existing patterns
- **Full Text Indexing**: Extracted text is indexed for search

### Dependencies
- `tesseract.js` (^5.0.4) - Installed

### API

#### Enable OCR During Upload
```javascript
const formData = new FormData();
formData.append('pdf', file);
formData.append('enableOcr', 'true');
formData.append('name', 'Pattern Name');

fetch('/api/patterns', {
  method: 'POST',
  body: formData
});
```

#### Run OCR on Existing Pattern
```javascript
fetch('/api/patterns/{patternId}/ocr', {
  method: 'POST'
});
```

#### Response
```json
{
  "success": true,
  "extractedText": "Full text extracted from the pattern..."
}
```

### Database
- **Field**: `extracted_text` (TEXT)
- **Flag**: `ocr_processed` (BOOLEAN)

### Frontend UI Suggestions
- Add toggle for "Enable OCR" in upload form
- Show "Extract Text" button in pattern details
- Display extracted text in a collapsible section

---

## Thread & Material Inventory

### Overview
Expand Yarnl beyond crochet patterns. New inventory tables for threads and materials support managing related items.

### Tables

#### Threads Table
```sql
threads (
  id SERIAL PRIMARY KEY,
  user_id: user this belongs to,
  name: thread name,
  brand: manufacturer,
  color_name: color description,
  color_hex: hex color code,
  thread_type: embroidery, sewing, etc.,
  weight: thread weight designation,
  length_meters: total length,
  quantity: number of spools/items,
  needle_size: compatible needle size,
  is_favorite: boolean,
  rating: 0-5 rating,
  notes: user notes,
  thumbnail: image reference
)
```

#### Materials Table
```sql
materials (
  id SERIAL PRIMARY KEY,
  user_id: user this belongs to,
  name: material name,
  category: clothing trim, stiffening, tools, etc.,
  description: detailed description,
  quantity: amount in stock,
  unit: item, meters, grams, etc.,
  color: color description,
  is_favorite: boolean,
  rating: 0-5 rating,
  notes: user notes,
  thumbnail: image reference,
  barcode_value: for scanning
)
```

### API Endpoints

#### Threads
- `GET /api/threads` - List all threads
- `POST /api/threads` - Create thread
- `PUT /api/threads/{id}` - Update thread
- `DELETE /api/threads/{id}` - Delete thread

#### Materials
- `GET /api/materials` - List all materials
- `POST /api/materials` - Create material
- `PUT /api/materials/{id}` - Update material
- `DELETE /api/materials/{id}` - Delete material

### Example Usage
```javascript
// Create a thread
const thread = {
  name: "DMC Pearl Cotton",
  brand: "DMC",
  colorName: "Red",
  colorHex: "#FF0000",
  threadType: "embroidery",
  weight: "#5",
  lengthMeters: 100,
  quantity: 1,
  needleSize: "7-8"
};

fetch('/api/threads', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(thread)
});
```

### Junction Tables
- `pattern_threads` - Link patterns to threads
- `pattern_materials` - Link patterns to materials

---

## Expanded Pattern Metadata

### New Fields
Additional metadata fields were added to patterns for better organization and planning:

| Field | Type | Description |
|-------|------|-------------|
| `inventory_id` | VARCHAR | Unique pattern ID |
| `file_type` | VARCHAR | pdf, image, or markdown |
| `needle_size` | VARCHAR | Recommended needle/hook size |
| `yarn_weight` | VARCHAR | Yarn weight category |
| `yardage_required` | NUMERIC | Yardage needed |
| `time_estimate_hours` | INTEGER | Estimated completion time |
| `skill_level` | VARCHAR | Beginner, Intermediate, Advanced, Expert |
| `size_range` | VARCHAR | "XS-L", "One Size", etc. |
| `designer_name` | VARCHAR | Pattern designer name |
| `source_url` | TEXT | Original pattern source |
| `extracted_text` | TEXT | OCR-extracted text |
| `ocr_processed` | BOOLEAN | Whether OCR has been run |

### Upload Form Integration
When uploading/creating patterns, include these optional fields:

```javascript
{
  name: "My Amigurumi",
  category: "Amigurumi",
  description: "A cute stuffed animal",
  // NEW FIELDS:
  needleSize: "4.0mm",
  yarnWeight: "Worsted",
  yardageRequired: 350,
  timeEstimateHours: 8,
  skillLevel: "Intermediate",
  sizeRange: "6 inches",
  designerName: "Original Design",
  sourceUrl: "https://example.com/pattern",
  enableOcr: true,
  generateBarcode: true
}
```

---

## Barcode Support

### Overview
Barcode support enables quick pattern identification and scanning. Each pattern can have a barcode generated automatically.

### Features
- **Automatic Generation**: Inventory IDs are converted to CODE128 barcodes
- **QR Codes**: Can be used for linking to patterns
- **Custom Barcodes**: Support for existing product barcodes
- **Barcode Database**: Central registry for all barcodes

### Database Structures

#### Patterns Barcode Fields
- `barcode_value`: The barcode value (text)
- `barcode_format`: Format type (CODE128, QR, etc.)
- `barcode_image`: Path to barcode image file

#### Barcode Database Table
```sql
barcode_database (
  id SERIAL PRIMARY KEY,
  user_id: pattern owner,
  barcode_value: the barcode string,
  item_type: 'pattern', 'material', etc.,
  item_id: ID of referenced item,
  item_name: name of item,
  is_custom_barcode: if user-created,
  database_source: internal tracking
)
```

### API Endpoints

#### Generate Barcode for Pattern
```javascript
fetch('/api/patterns/{id}/barcode', {
  method: 'POST'
})
// Response: { barcode: "PAT-1A2B3C-F4G5H6JK" }
```

#### Scan Barcode
```javascript
fetch('/api/barcode/scan', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ barcodeValue: "PAT-1A2B3C-F4G5H6JK" })
})
// Response: { found: true, type: 'pattern', item: {...} }
```

### Frontend Implementation
- Add input field for barcode scanner
- Display barcode (or QR code) in pattern details
- Enable barcode scanning workflow

### Barcode Format
- **Format**: CODE128 (default)
- **Value**: Inventory ID (e.g., `PAT-1A2B3C-F4G5H6JK`)
- **Generation**: Can be generated on-demand in frontend using jsbarcode library

---

## Threadloop Integration

### Overview
Connect your Yarnl patterns with Threadloop for community sharing and pattern discovery.

### Setup

#### Prerequisites
1. Threadloop account
2. Threadloop API key

#### Configuration
```javascript
// Save Threadloop credentials
fetch('/api/threadloop/settings', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    apiKey: "your_threadloop_api_key",
    username: "your_threadloop_username"
  })
});
```

### API Endpoints

#### Fetch Threadloop Patterns
```javascript
fetch('/api/threadloop/patterns', {
  method: 'GET'
})
// Response: Array of patterns available on Threadloop
```

#### Import Pattern from Threadloop
```javascript
fetch('/api/threadloop/import', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    threadloopPatternId: "pattern_id"
  })
})
// Response: Newly created pattern in Yarnl
```

### Pattern Link
Patterns imported from Threadloop are marked with:
- `threadloop_url`: Link to original pattern
- `threadloop_id`: Original pattern ID

### Frontend UI Suggestions
- "Import from Threadloop" button in library
- "Threadloop Settings" in admin panel
- Display Threadloop URL badge on imported patterns

### Note
The Threadloop API integration is a placeholder. Actual implementation requires:
1. Current Threadloop API documentation
2. API endpoint verification
3. Authentication flow testing

---

## Database Migration Notes

### New Tables Created
```sql
CREATE TABLE threads (...)
CREATE TABLE materials (...)
CREATE TABLE pattern_threads (...)
CREATE TABLE pattern_materials (...)
CREATE TABLE barcode_database (...)
```

### Existing Tables Modified
```sql
ALTER TABLE patterns ADD COLUMN inventory_id VARCHAR UNIQUE
ALTER TABLE patterns ADD COLUMN file_type VARCHAR DEFAULT 'pdf'
ALTER TABLE patterns ADD COLUMN needle_size VARCHAR
ALTER TABLE patterns ADD COLUMN yarn_weight VARCHAR
ALTER TABLE patterns ADD COLUMN yardage_required NUMERIC
ALTER TABLE patterns ADD COLUMN time_estimate_hours INTEGER
ALTER TABLE patterns ADD COLUMN skill_level VARCHAR
ALTER TABLE patterns ADD COLUMN size_range VARCHAR
ALTER TABLE patterns ADD COLUMN designer_name VARCHAR
ALTER TABLE patterns ADD COLUMN source_url TEXT
ALTER TABLE patterns ADD COLUMN extracted_text TEXT
ALTER TABLE patterns ADD COLUMN ocr_processed BOOLEAN
ALTER TABLE patterns ADD COLUMN barcode_value VARCHAR UNIQUE
ALTER TABLE patterns ADD COLUMN barcode_format VARCHAR
ALTER TABLE patterns ADD COLUMN barcode_image VARCHAR
ALTER TABLE patterns ADD COLUMN threadloop_url TEXT
ALTER TABLE patterns ADD COLUMN threadloop_id VARCHAR

ALTER TABLE users ADD COLUMN threadloop_api_key TEXT
ALTER TABLE users ADD COLUMN threadloop_username VARCHAR
```

All migrations are automatic and idempotent - they only create tables or add columns if they don't already exist.

---

## Frontend Implementation Checklist

### Pattern Upload Form
- [ ] Add file type selector (PDF/Image)
- [ ] Add extended metadata fields
- [ ] Add OCR enable toggle
- [ ] Add barcode generation toggle
- [ ] Show inventory ID in confirmation

### Pattern Details View
- [ ] Display inventory ID
- [ ] Show barcode (if exists)
- [ ] Display all metadata fields
- [ ] Show "Extract Text" OCR button
- [ ] Display OCR extracted text (if available)

### New UI Tabs/Sections
- [ ] Threads inventory tab
- [ ] Materials inventory tab
- [ ] Thread-pattern linking UI
- [ ] Material-pattern linking UI

### Barcode Scanning
- [ ] Add barcode scanner input
- [ ] Handle barcode scan results
- [ ] Display quick-link on scan success

### Threadloop Integration
- [ ] Add Threadloop settings panel
- [ ] Add "Import from Threadloop" workflow
- [ ] Display Threadloop links on imported patterns

---

## Dependencies

### New NPM Packages
```bash
npm install tesseract.js@^5.0.4
npm install jsbarcode@^3.11.5
npm install axios@^1.6.2
npm install uuid@^9.0.1
```

### Server Side
- **tesseract.js**: OCR processing
- **jsbarcode**: Barcode generation
- **axios**: HTTP client for external APIs
- **uuid**: Unique ID generation

### Frontend (Optional)
- **jsbarcode**: Client-side barcode generation
- **quagga.js**: Barcode scanning (optional, for hardware scanner integration)

---

## Error Handling

### Common Issues

#### OCR Processing Fails
- Ensure tessdata files are available
- Check image quality
- Verify file is not corrupted

#### Threadloop Connection Fails
- Verify API key is correct
- Check internet connection
- Review Threadloop API status

#### Barcode Scan Not Found
- Verify barcode value
- Check it was registered in database
- Ensure user owns the pattern

---

## Security Considerations

1. **Barcode Sensitivity**: Barcodes are user-specific; ensure proper ACL
2. **OCR Processing**: Large files may consume resources; rate-limit OCR requests
3. **External APIs**: Store API keys securely; use environment variables
4. **File Upload Validation**: Verify file integrity and type

---

## Performance Tips

1. **OCR Processing**: Run asynchronously; consider background job queue
2. **Thumbnail Generation**: Cache generated thumbnails
3. **Barcode Lookup**: Index barcode_database for quick scans
4. **API Calls**: Implement caching for Threadloop API responses

---

## Future Enhancements

1. **Bulk Barcode Generation**: Generate barcodes for multiple patterns
2. **Mobile Barcode Scanner**: Native camera-based scanning
3. **Advanced OCR**: Support multi-language OCR
4. **Material Usage Tracking**: Track material consumption per project
5. **Threadloop Sync**: Two-way sync with Threadloop
6. **Custom Barcode Formats**: QR codes, UPC-A, EAN, etc.

---

## Support & Troubleshooting

For issues or questions:
1. Check logs: `docker logs yarnl`
2. Verify database migrations: Check PostgreSQL schema
3. Test API endpoints directly: Use curl or Postman
4. Review browser console for frontend errors

