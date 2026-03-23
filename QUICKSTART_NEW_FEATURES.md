# Quick Start Guide - New Features

## What's New

Yarnl has been enhanced with several powerful features for pattern and inventory management:

1. **Unique Pattern IDs** - Automatic inventory numbering for your patterns
2. **Image Upload** - Upload JPG, PNG, WebP, TIFF in addition to PDFs
3. **OCR Support** - Extract text from images and PDFs automatically
4. **Thread Inventory** - Manage your thread collection
5. **Material Inventory** - Track materials, trims, and supplies
6. **Expanded Metadata** - Add needle size, yarn weight, designer, and more
7. **Barcode Support** - Generate and scan barcodes for quick identification
8. **Threadloop Integration** - Import patterns from Threadloop

---

## Getting Started

### Installation

1. **Update dependencies**:
```bash
npm install
```

2. **Restart Yarnl**:
```bash
docker compose restart yarnl
# or
npm run dev  # for development
```

3. **Database migrations run automatically** on startup

### First Steps

#### 1. Upload a Pattern with Enhanced Features
- Go to "Upload Patterns" → "PDF"
- Fill in the basic info and optional metadata:
  - Needle Size
  - Yarn Weight
  - Yardage Required
  - Time Estimate
  - Designer Name
  - Source URL
- Check "Extract text from image (OCR)"
- Check "Generate barcode"
- Click "Upload All"

#### 2. View Pattern Details
- Click on your uploaded pattern
- You'll see:
  - Inventory ID (e.g., `PAT-1A2B3C-F4G5H6JK`)
  - Barcode (if generated)
  - All metadata fields
  - Extracted text (if OCR was enabled)

#### 3. Try Barcode Scanning
- Scroll to "Scan Barcode" section
- Type or scan the barcode value
- Tap Enter to lookup the item

#### 4. Create Thread Inventory
- Click "Threads" tab
- Click "+ Add Thread"
- Fill in thread details (name, brand, color, type)
- Click "Save Thread"

#### 5. Create Material Inventory
- Click "Materials" tab
- Click "+ Add Material"
- Fill in material details (name, category, quantity)
- Click "Save Material"

---

## API Usage Examples

### Upload Pattern with All Features
```bash
curl -X POST http://localhost:3000/api/patterns \
  -F "pdf=@pattern.jpg" \
  -F "name=My Amigurumi" \
  -F "category=Amigurumi" \
  -F "needleSize=4.0mm" \
  -F "yarnWeight=Worsted" \
  -F "yardageRequired=350" \
  -F "timeEstimateHours=8" \
  -F "skillLevel=Intermediate" \
  -F "enableOcr=true" \
  -F "generateBarcode=true"
```

### Get Pattern OCR Text
```bash
curl -X POST http://localhost:3000/api/patterns/123/ocr
```

### Create Thread
```bash
curl -X POST http://localhost:3000/api/threads \
  -H "Content-Type: application/json" \
  -d '{
    "name": "DMC Pearl Cotton",
    "brand": "DMC",
    "colorHex": "#FF0000",
    "threadType": "embroidery",
    "weight": "#5",
    "lengthMeters": 100
  }'
```

### Scan Barcode
```bash
curl -X POST http://localhost:3000/api/barcode/scan \
  -H "Content-Type: application/json" \
  -d '{"barcodeValue": "PAT-1A2B3C-F4G5H6JK"}'
```

---

## Documentation Files

- **FEATURES_NEW.md** - Detailed feature documentation
- **FRONTEND_INTEGRATION.md** - Frontend integration guide with code examples
- **IMPLEMENTATION_SUMMARY.md** - Technical implementation details

---

## Troubleshooting

### OCR Not Working
- Check that Tesseract WASM files are accessible
- Try with a high-quality image
- Check server logs for errors

### Barcode Not Generating
- Ensure jsbarcode library is installed
- Check that pattern has valid inventory_id
- Try generating from pattern details page

### Image Upload Fails
- Verify file is valid JPEG/PNG/WebP/TIFF
- Check file size (max 100MB)
- Try with a different format

### Threads/Materials Not Showing
- Refresh the page
- Check browser console for errors
- Verify you're logged in

### Threadloop Not Working
- Add your API key in settings
- Verify Threadloop API is accessible
- Check credentials are correct

---

## Example Workflows

### Workflow 1: Upload Pattern with Full Details
1. Prepare pattern image/PDF
2. Open Yarnl → Upload Patterns
3. Enter pattern name and category
4. Fill in metadata fields
5. Enable OCR and barcode
6. Upload
7. View pattern details with inventory ID and barcode

### Workflow 2: Manage Thread Inventory
1. Organize your threads by color, type, brand
2. Add each thread to inventory
3. Set favorites and ratings
4. Link threads to patterns that use them
5. Use for pattern planning

### Workflow 3: Quick Pattern Lookup by Barcode
1. Use barcode scanner (physical or software)
2. Scan pattern barcode
3. System instantly retrieves pattern details
4. View all relevant information

### Workflow 4: Import from Threadloop
1. Configure Threadloop API key
2. Browse available patterns
3. Click Import
4. Pattern appears in your library
5. Update with local metadata

---

## File Type Support

### Images
- JPEG (.jpg, .jpeg) ✓
- PNG (.png) ✓
- WebP (.webp) ✓
- TIFF (.tiff) ✓

### Documents
- PDF (.pdf) ✓

### Text
- Markdown (.md) ✓

---

## Sample Data

### Thread Example
```json
{
  "name": "DMC Stranded Cotton",
  "brand": "DMC",
  "colorName": "Red",
  "colorHex": "#FF0000",
  "threadType": "embroidery",
  "weight": "#6",
  "lengthMeters": 100,
  "quantity": 1,
  "needleSize": "7-8"
}
```

### Material Example
```json
{
  "name": "Fabric Stiffener",
  "category": "Finishing",
  "description": "For stiffening fabric pieces",
  "quantity": 1,
  "unit": "bottle",
  "color": "Clear"
}
```

### Pattern Example
```json
{
  "name": "Amigurumi Cat",
  "category": "Amigurumi",
  "needleSize": "3.5mm",
  "yarnWeight": "DK",
  "yardageRequired": 250,
  "timeEstimateHours": 6,
  "skillLevel": "Intermediate",
  "designerName": "Jane Smith",
  "sourceUrl": "https://example.com/pattern"
}
```

---

## API Endpoint Quick Reference

### Patterns
- `POST /api/patterns` - Upload pattern
- `POST /api/patterns/{id}/ocr` - Extract text

### Threads
- `GET /api/threads` - List
- `POST /api/threads` - Create
- `PUT /api/threads/{id}` - Update
- `DELETE /api/threads/{id}` - Delete

### Materials
- `GET /api/materials` - List
- `POST /api/materials` - Create
- `PUT /api/materials/{id}` - Update
- `DELETE /api/materials/{id}` - Delete

### Barcodes
- `POST /api/patterns/{id}/barcode` - Generate
- `POST /api/barcode/scan` - Lookup

### Threadloop
- `POST /api/threadloop/settings` - Configure
- `GET /api/threadloop/patterns` - Fetch
- `POST /api/threadloop/import` - Import

---

## Tips & Tricks

1. **Inventory IDs are Automatic** - Don't worry about numbering, Yarnl handles it
2. **OCR Works Better with Quality Images** - High-res images give better results
3. **Barcode Scanning** - Use QR code generators to create QR barcodes for faster scanning
4. **Thread Colors** - Use the color picker to set thread colors for visual display
5. **Material Categories** - Create consistent naming for material categories for easier filtering
6. **Metadata Search** - OCR text is searchable, making patterns easier to find

---

## Keyboard Shortcuts for Barcode Scanner

- **Enter** - Scan with typed/scanned barcode
- **Escape** - Clear scanner input

---

## Next Steps

1. Read FEATURES_NEW.md for detailed documentation
2. Review FRONTEND_INTEGRATION.md for UI implementation
3. Check IMPLEMENTATION_SUMMARY.md for technical details
4. Test each feature with sample data
5. Update frontend UI to match your preferences
6. Configure Threadloop (optional)

---

## Support

For issues or questions:
1. Check the documentation files
2. Review server logs: `docker logs yarnl`
3. Check browser console for errors
4. Verify database status: `docker exec yarnl-db psql -U yarnl -d yarnl -c "\\dt"`

---

## Changelog

### Version 1.0.0 - New Features Release

#### Added
- ✨ Unique Pattern IDs for inventory numbering
- ✨ Image upload support (JPEG, PNG, WebP, TIFF)
- ✨ OCR integration for text extraction
- ✨ Thread inventory management
- ✨ Material inventory management
- ✨ Expanded pattern metadata fields
- ✨ Barcode generation and scanning
- ✨ Threadloop API integration

#### Improved
- 📈 File upload limit increased to 100MB
- 📈 Better file type detection
- 📈 Auto-generated thumbnails for all image types

#### Technical
- 🔧 5 new database tables
- 🔧 16 new database columns
- 🔧 15 new API endpoints
- 🔧 4 new npm dependencies

---

## License

Yarnl - Self-hosted pattern management for crafters
MIT License

