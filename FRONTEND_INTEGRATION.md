# Yarnl Frontend Integration Guide

This guide shows how to integrate the new features into the Yarnl frontend UI.

## Quick Start

### 1. Extended Pattern Upload Form

Update the pattern upload form in `public/index.html` to include new fields:

```html
<!-- In the upload panel, add extended metadata section -->
<div class="form-group">
    <label>Pattern Details</label>
    
    <!-- File Type -->
    <div class="form-row">
        <input type="text" placeholder="Needle Size (e.g., 4.0mm)" id="needleSize" />
        <input type="text" placeholder="Yarn Weight" id="yarnWeight" />
    </div>
    
    <!-- Yardage & Time -->
    <div class="form-row">
        <input type="number" placeholder="Yardage Required" id="yardageRequired" step="0.1" />
        <input type="number" placeholder="Time (hours)" id="timeEstimateHours" />
    </div>
    
    <!-- Designer & Source -->
    <div class="form-row">
        <input type="text" placeholder="Designer Name" id="designerName" />
        <input type="url" placeholder="Source URL" id="sourceUrl" />
    </div>
    
    <!-- Size & Skill -->
    <div class="form-row">
        <input type="text" placeholder="Size Range (e.g., XS-L)" id="sizeRange" />
        <select id="skillLevel">
            <option value="">Select Skill Level</option>
            <option value="Beginner">Beginner</option>
            <option value="Intermediate">Intermediate</option>
            <option value="Advanced">Advanced</option>
            <option value="Expert">Expert</option>
        </select>
    </div>
    
    <!-- New Features Toggles -->
    <div class="form-row">
        <label>
            <input type="checkbox" id="enableOcr" />
            Extract text from image (OCR)
        </label>
        <label>
            <input type="checkbox" id="generateBarcode" checked />
            Generate barcode
        </label>
    </div>
</div>
```

### 2. JavaScript Upload Handler

Update the upload handler in `public/app.js`:

```javascript
async function uploadPattern(file) {
    const formData = new FormData();
    
    // File and basic info
    formData.append('pdf', file);
    formData.append('name', document.getElementById('patternName').value);
    formData.append('category', document.getElementById('patternCategory').value);
    formData.append('description', document.getElementById('patternDescription').value);
    
    // NEW: Extended metadata
    formData.append('needleSize', document.getElementById('needleSize').value);
    formData.append('yarnWeight', document.getElementById('yarnWeight').value);
    formData.append('yardageRequired', document.getElementById('yardageRequired').value);
    formData.append('timeEstimateHours', document.getElementById('timeEstimateHours').value);
    formData.append('designerName', document.getElementById('designerName').value);
    formData.append('sourceUrl', document.getElementById('sourceUrl').value);
    formData.append('sizeRange', document.getElementById('sizeRange').value);
    formData.append('skillLevel', document.getElementById('skillLevel').value);
    
    // NEW: Feature toggles
    formData.append('enableOcr', document.getElementById('enableOcr').checked);
    formData.append('generateBarcode', document.getElementById('generateBarcode').checked);
    
    try {
        const response = await fetch('/api/patterns', {
            method: 'POST',
            body: formData
        });
        
        if (!response.ok) throw new Error('Upload failed');
        
        const pattern = await response.json();
        console.log('Pattern uploaded:', pattern);
        
        // Show inventory ID
        if (pattern.inventory_id) {
            alert(`Pattern created with ID: ${pattern.inventory_id}`);
        }
        
        // Refresh pattern list
        loadPatterns();
        
    } catch (error) {
        console.error('Error uploading pattern:', error);
        alert('Error uploading pattern: ' + error.message);
    }
}
```

### 3. Pattern Details Display

Show the new fields in pattern detail view:

```javascript
function displayPatternDetails(pattern) {
    const details = `
        <div class="pattern-info">
            <!-- Inventory ID -->
            ${pattern.inventory_id ? `
                <div class="info-row">
                    <span class="label">Inventory ID:</span>
                    <span class="value">${pattern.inventory_id}</span>
                    <button onclick="copyToClipboard('${pattern.inventory_id}')">Copy</button>
                </div>
            ` : ''}
            
            <!-- File Type -->
            <div class="info-row">
                <span class="label">Type:</span>
                <span class="value">${(pattern.file_type || 'pdf').toUpperCase()}</span>
            </div>
            
            <!-- Metadata -->
            ${pattern.needle_size ? `
                <div class="info-row">
                    <span class="label">Needle Size:</span>
                    <span class="value">${pattern.needle_size}</span>
                </div>
            ` : ''}
            
            ${pattern.yarn_weight ? `
                <div class="info-row">
                    <span class="label">Yarn Weight:</span>
                    <span class="value">${pattern.yarn_weight}</span>
                </div>
            ` : ''}
            
            ${pattern.yardage_required ? `
                <div class="info-row">
                    <span class="label">Yardage:</span>
                    <span class="value">${pattern.yardage_required} yards</span>
                </div>
            ` : ''}
            
            ${pattern.time_estimate_hours ? `
                <div class="info-row">
                    <span class="label">Est. Time:</span>
                    <span class="value">${pattern.time_estimate_hours} hours</span>
                </div>
            ` : ''}
            
            ${pattern.skill_level ? `
                <div class="info-row">
                    <span class="label">Skill Level:</span>
                    <span class="value">${pattern.skill_level}</span>
                </div>
            ` : ''}
            
            <!-- Barcode -->
            ${pattern.barcode_value ? `
                <div class="info-row barcode-section">
                    <span class="label">Barcode:</span>
                    <svg id="barcode-${pattern.id}"></svg>
                    <script>
                        JsBarcode("#barcode-${pattern.id}", "${pattern.barcode_value}", {
                            format: "CODE128",
                            width: 2,
                            height: 50,
                            margin: 10
                        });
                    </script>
                </div>
            ` : ''}
            
            <!-- OCR Text -->
            ${pattern.extracted_text ? `
                <div class="info-row ocr-section">
                    <span class="label">Extracted Text:</span>
                    <details>
                        <summary>View text (${pattern.extracted_text.length} characters)</summary>
                        <pre>${escapeHtml(pattern.extracted_text)}</pre>
                    </details>
                </div>
            ` : ''}
            
            <!-- External Links -->
            ${pattern.source_url ? `
                <div class="info-row">
                    <span class="label">Source:</span>
                    <a href="${pattern.source_url}" target="_blank">${pattern.source_url}</a>
                </div>
            ` : ''}
            
            ${pattern.threadloop_url ? `
                <div class="info-row">
                    <span class="label">Threadloop:</span>
                    <a href="${pattern.threadloop_url}" target="_blank">View on Threadloop</a>
                </div>
            ` : ''}
        </div>
    `;
    
    return details;
}
```

### 4. OCR Button

Add a button to extract text from existing patterns:

```javascript
async function extractPatternText(patternId) {
    try {
        const response = await fetch(`/api/patterns/${patternId}/ocr`, {
            method: 'POST'
        });
        
        if (!response.ok) throw new Error('OCR failed');
        
        const result = await response.json();
        console.log('Text extracted:', result.extractedText);
        
        // Show extracted text
        alert('Text extraction complete. Refresh to see results.');
        
        // Refresh pattern details
        const pattern = await fetchPattern(patternId);
        displayPatternDetails(pattern);
        
    } catch (error) {
        console.error('OCR error:', error);
        alert('Error extracting text: ' + error.message);
    }
}

// Add button to pattern details
const ocrButton = pattern.ocr_processed ? 
    '<button disabled>OCR Complete</button>' :
    `<button onclick="extractPatternText(${pattern.id})">Extract Text</button>`;
```

---

## Threads & Materials UI

### 5. Add Threads Tab

```html
<!-- Add to main navigation -->
<button class="nav-item" id="threads-tab" onclick="showThreads()">
    <svg><!-- thread icon --></svg>
    Threads
</button>

<!-- Threads panel -->
<div id="threads-panel" class="panel" style="display: none;">
    <h2>Thread Inventory</h2>
    <button onclick="showThreadForm()" class="btn btn-primary">+ Add Thread</button>
    
    <div id="threads-list" class="items-grid">
        <!-- Threads will be loaded here -->
    </div>
</div>

<!-- Thread form modal -->
<div id="thread-form-modal" class="modal" style="display: none;">
    <form id="thread-form">
        <input type="text" placeholder="Thread Name" id="thread-name" required />
        <input type="text" placeholder="Brand" id="thread-brand" />
        <input type="text" placeholder="Color Name" id="thread-color-name" />
        <input type="color" placeholder="Color" id="thread-color-hex" />
        <input type="text" placeholder="Type (embroidery, sewing, etc.)" id="thread-type" />
        <input type="text" placeholder="Weight" id="thread-weight" />
        <input type="number" placeholder="Length (meters)" id="thread-length" />
        <input type="number" placeholder="Quantity" id="thread-quantity" value="1" />
        <input type="text" placeholder="Compatible Needle Size" id="thread-needle-size" />
        <textarea placeholder="Notes" id="thread-notes"></textarea>
        
        <button type="submit" class="btn btn-primary">Save Thread</button>
        <button type="button" onclick="closeThreadForm()" class="btn btn-secondary">Cancel</button>
    </form>
</div>
```

### 6. Load and Display Threads

```javascript
async function loadThreads() {
    try {
        const response = await fetch('/api/threads');
        const threads = await response.json();
        
        const container = document.getElementById('threads-list');
        container.innerHTML = threads.map(thread => `
            <div class="thread-card">
                <div class="color-swatch" style="background-color: ${thread.color_hex || '#ccc'}"></div>
                <h3>${thread.name}</h3>
                <p>${thread.brand || 'No brand'}</p>
                <small>${thread.type || ''}</small>
                <div class="controls">
                    <button onclick="editThread(${thread.id})">Edit</button>
                    <button onclick="deleteThread(${thread.id})">Delete</button>
                </div>
            </div>
        `).join('');
        
    } catch (error) {
        console.error('Error loading threads:', error);
    }
}

async function addThread(formData) {
    try {
        const response = await fetch('/api/threads', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                name: formData.get('thread-name'),
                brand: formData.get('thread-brand'),
                colorName: formData.get('thread-color-name'),
                colorHex: formData.get('thread-color-hex'),
                threadType: formData.get('thread-type'),
                weight: formData.get('thread-weight'),
                lengthMeters: parseInt(formData.get('thread-length')),
                quantity: parseInt(formData.get('thread-quantity')),
                needleSize: formData.get('thread-needle-size'),
                notes: formData.get('thread-notes')
            })
        });
        
        if (!response.ok) throw new Error('Failed to add thread');
        
        closeThreadForm();
        loadThreads();
        
    } catch (error) {
        console.error('Error adding thread:', error);
        alert('Error: ' + error.message);
    }
}
```

### 7. Add Materials Tab (Similar to Threads)

```html
<!-- Materials tab -->
<button class="nav-item" id="materials-tab" onclick="showMaterials()">
    <svg><!-- materials icon --></svg>
    Materials
</button>

<!-- Materials panel -->
<div id="materials-panel" class="panel" style="display: none;">
    <h2>Materials Inventory</h2>
    <button onclick="showMaterialForm()" class="btn btn-primary">+ Add Material</button>
    
    <div id="materials-list" class="items-grid">
        <!-- Materials will be loaded here -->
    </div>
</div>
```

---

## Barcode Scanning UI

### 8. Add Barcode Scanner

```html
<!-- In main interface -->
<div id="barcode-scanner-section" class="scanner-section">
    <h3>Scan Barcode</h3>
    <input type="text" 
           id="barcode-input" 
           placeholder="Scan barcode or enter ID"
           onkeypress="handleBarcodeScan(event)" />
    <div id="scan-result" class="scan-result" style="display: none;">
        <!-- Results will appear here -->
    </div>
</div>
```

### 9. Barcode Scanning Handler

```javascript
async function handleBarcodeScan(event) {
    if (event.key !== 'Enter') return;
    
    const barcodeValue = document.getElementById('barcode-input').value.trim();
    if (!barcodeValue) return;
    
    try {
        const response = await fetch('/api/barcode/scan', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ barcodeValue })
        });
        
        const result = await response.json();
        const resultDiv = document.getElementById('scan-result');
        
        if (result.found) {
            const item = result.item;
            resultDiv.innerHTML = `
                <div class="success">
                    <strong>${result.type === 'pattern' ? 'Pattern' : 'Material'} Found!</strong>
                    <p>${item.name}</p>
                    <button onclick="openItem('${result.type}', ${item.id})">View</button>
                </div>
            `;
        } else {
            resultDiv.innerHTML = '<div class="error">Barcode not found</div>';
        }
        
        resultDiv.style.display = 'block';
        document.getElementById('barcode-input').value = '';
        
        // Clear result after 3 seconds
        setTimeout(() => {
            resultDiv.style.display = 'none';
        }, 3000);
        
    } catch (error) {
        console.error('Barcode scan error:', error);
        alert('Error scanning barcode: ' + error.message);
    }
}
```

---

## Threadloop Integration UI

### 10. Add Threadloop Settings

```html
<!-- In admin panel -->
<div id="threadloop-settings" class="settings-section">
    <h3>Threadloop Integration</h3>
    <div class="form-group">
        <label>API Key</label>
        <input type="password" id="threadloop-api-key" placeholder="Enter Threadloop API key" />
    </div>
    <div class="form-group">
        <label>Username</label>
        <input type="text" id="threadloop-username" placeholder="Your Threadloop username" />
    </div>
    <button onclick="saveThreadloopSettings()" class="btn btn-primary">Save Settings</button>
</div>
```

### 11. Threadloop Import UI

```html
<!-- Add to library panel -->
<button onclick="showThreadloopImport()" class="btn btn-secondary">
    Import from Threadloop
</button>

<!-- Threadloop import modal -->
<div id="threadloop-import-modal" class="modal" style="display: none;">
    <h3>Import from Threadloop</h3>
    <div id="threadloop-patterns-list">
        <!-- Loading... -->
    </div>
</div>
```

### 12. Threadloop Import Handler

```javascript
async function loadThreadloopPatterns() {
    try {
        const response = await fetch('/api/threadloop/patterns');
        const patterns = await response.json();
        
        const container = document.getElementById('threadloop-patterns-list');
        container.innerHTML = patterns.map(pattern => `
            <div class="threadloop-pattern-card">
                <h4>${pattern.name}</h4>
                <p>${pattern.description || 'No description'}</p>
                <button onclick="importThreadloopPattern(${pattern.id})">
                    Import
                </button>
            </div>
        `).join('');
        
    } catch (error) {
        console.error('Error loading Threadloop patterns:', error);
    }
}

async function importThreadloopPattern(threadloopPatternId) {
    try {
        const response = await fetch('/api/threadloop/import', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ threadloopPatternId })
        });
        
        if (!response.ok) throw new Error('Import failed');
        
        const pattern = await response.json();
        alert(`Pattern "${pattern.name}" imported successfully!`);
        
        // Refresh pattern list
        loadPatterns();
        closeThreadloopModal();
        
    } catch (error) {
        console.error('Threadloop import error:', error);
        alert('Error importing pattern: ' + error.message);
    }
}
```

---

## CSS Styling Tips

Add these styles to `public/styles.css`:

```css
/* Inventory ID badge */
.inventory-id {
    display: inline-block;
    background: #f0f0f0;
    padding: 4px 8px;
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.85em;
}

/* Barcode styling */
.barcode-section {
    margin: 1rem 0;
    padding: 1rem;
    background: #f9f9f9;
    border-radius: 4px;
}

#barcode-input {
    border: 2px solid #ddd;
    padding: 12px;
    font-size: 1.1em;
    width: 100%;
    max-width: 400px;
}

.scan-result {
    margin-top: 1rem;
    padding: 1rem;
    border-radius: 4px;
}

.scan-result.success {
    background: #d4edda;
    border: 1px solid #c3e6cb;
    color: #155724;
}

.scan-result.error {
    background: #f8d7da;
    border: 1px solid #f5c6cb;
    color: #721c24;
}

/* Thread/Material cards */
.thread-card, .material-card {
    background: white;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1rem;
}

.color-swatch {
    width: 40px;
    height: 40px;
    border-radius: 4px;
    margin-bottom: 0.5rem;
    border: 1px solid #ddd;
}

/* OCR text section */
.ocr-section {
    background: #f5f5f5;
    padding: 1rem;
    border-radius: 4px;
    margin: 1rem 0;
}

.ocr-section pre {
    background: white;
    padding: 0.5rem;
    border-radius: 4px;
    max-height: 300px;
    overflow-y: auto;
}
```

---

## Integration Checklist

- [ ] Add extended metadata fields to upload form
- [ ] Update pattern display with new fields
- [ ] Add OCR button and handler
- [ ] Add barcode generation/display
- [ ] Add Threads tab and CRUD UI
- [ ] Add Materials tab and CRUD UI
- [ ] Add barcode scanner input and handler
- [ ] Add Threadloop settings panel
- [ ] Add Threadloop import UI
- [ ] Test all new features
- [ ] Update responsive design for mobile
- [ ] Add loading states and error handling

---

## Testing Tips

1. **Upload Test**: Upload a PDF and an image with metadata
2. **OCR Test**: Enable OCR during upload and verify text extraction
3. **Barcode Test**: Verify barcode generation and scanning
4. **Thread Test**: Create/edit/delete threads
5. **Threadloop Test**: Test with real/mock API credentials

