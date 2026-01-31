# Update Form-Data — Read Spreadsheet and Submit via POSTMAN

A small example project demonstrating how to read rows from a spreadsheet (CSV or XLSX) and use those rows to build multipart/form-data requests that can be sent from POSTMAN to update an API. Includes sample scripts (Node.js and Python), mapping guidance, and Postman workflow notes to help you automate spreadsheet-driven API updates.

---

## Contents
- Description
- Features
- Prerequisites
- Quickstart (Node.js and Python)
- Postman usage
- Example mapping
- Troubleshooting
- Contributing & License

---

## Description
This repository shows how to:
- extract rows from a spreadsheet (CSV or XLSX),
- map spreadsheet columns to form-data fields,
- construct multipart/form-data requests (including files/binaries),
- and send those requests via POSTMAN to update a target API endpoint.

It is intended as a reference for backend developers, QA engineers, and anyone automating API calls using spreadsheet data.

---

## Features
- Examples for both Node.js and Python to read spreadsheets and prepare form-data payloads
- Guidance for importing and running requests from POSTMAN
- Mapping examples for fields and files
- Minimal, easy-to-adapt code to fit your API schema

---

## Prerequisites
- Node.js (>= 14) and npm, or Python (>= 3.8) and pip
- POSTMAN (for running collections)
- Access to the target API endpoint and any required credentials (API key / bearer token)
- Spreadsheet file (CSV or XLSX)

---

## Quickstart — Node.js example

Install dependencies:
npm init -y
npm install axios form-data xlsx dotenv

Create a `.env` file:
API_URL=https://api.example.com/update
API_TOKEN=your_api_token_here

Example script (index.js):
```js
require('dotenv').config();
const fs = require('fs');
const path = require('path');
const axios = require('axios');
const FormData = require('form-data');
const XLSX = require('xlsx');

const workbook = XLSX.readFile('data.xlsx'); // or .csv via csv parser
const sheet = workbook.Sheets[workbook.SheetNames[0]];
const rows = XLSX.utils.sheet_to_json(sheet);

async function sendRowAsForm(row) {
  const form = new FormData();
  // Map spreadsheet columns to form fields
  form.append('id', row.id);
  form.append('name', row.name);
  form.append('email', row.email);

  // If there's a filepath column and you want to upload a file:
  if (row.attachment && fs.existsSync(row.attachment)) {
    form.append('attachment', fs.createReadStream(path.resolve(row.attachment)));
  }

  const headers = {
    ...form.getHeaders(),
    Authorization: `Bearer ${process.env.API_TOKEN}`,
  };

  const res = await axios.post(process.env.API_URL, form, { headers });
  return res.data;
}

(async () => {
  for (const row of rows) {
    try {
      const result = await sendRowAsForm(row);
      console.log('Updated:', row.id, result);
    } catch (err) {
      console.error('Error updating', row.id, err.response?.data || err.message);
    }
  }
})();
```

Run:
node index.js

---

## Quickstart — Python example

Install dependencies:
pip install requests openpyxl python-dotenv

Create a `.env` file:
API_URL=https://api.example.com/update
API_TOKEN=your_api_token_here

Example script (update_from_xlsx.py):
```py
import os
import requests
from openpyxl import load_workbook
from dotenv import load_dotenv

load_dotenv()
API_URL = os.getenv('API_URL')
API_TOKEN = os.getenv('API_TOKEN')

wb = load_workbook('data.xlsx')
ws = wb.active

# Assuming header row in the first row
headers = [cell.value for cell in ws[1]]

for row in ws.iter_rows(min_row=2, values_only=True):
    data = dict(zip(headers, row))
    files = {}
    # If your spreadsheet stores local file paths in column 'attachment'
    if data.get('attachment'):
        path = data['attachment']
        if os.path.exists(path):
            files['attachment'] = open(path, 'rb')

    payload = {
        'id': data.get('id'),
        'name': data.get('name'),
        'email': data.get('email'),
    }

    headers_req = {'Authorization': f'Bearer {API_TOKEN}'}
    try:
        resp = requests.post(API_URL, data=payload, files=files or None, headers=headers_req)
        resp.raise_for_status()
        print('Updated', data.get('id'), resp.json())
    except Exception as e:
        print('Error updating', data.get('id'), str(e))
    finally:
        if files:
            files['attachment'].close()
```

Run:
python update_from_xlsx.py

---

## POSTMAN usage

1. Prepare the request:
   - Method: POST (or appropriate method)
   - URL: your target endpoint (use a Postman environment variable like {{API_URL}})
   - Authorization: Bearer Token or other as required (store token in environment variable {{API_TOKEN}})

2. In Body tab:
   - Choose form-data
   - Add key-value pairs that match your API fields (e.g., id, name, email)
   - For file uploads, set the type to "File" and choose a file to send

3. Automating with data files:
   - Use "Runner" → Import data file (CSV/JSON) to run the same request with different rows
   - Map Postman request fields to columns in your CSV (use double-curly variables like {{id}}, {{name}} if using pre-request scripts or environment variables)

4. Importing a collection:
   - Create a collection and export it (or add sample requests to a collection in this repo)
   - Import into Postman via File → Import

Notes:
- If your spreadsheet contains file paths and you want to upload those files from Postman Runner, you must manually attach files per run iteration or use Postman’s file upload automation features (Postman data files do not upload local files automatically in all cases — check Postman Runner docs).

---

## Example mapping
Spreadsheet columns: id, name, email, attachment
Form-data mapping:
- id -> id (text)
- name -> name (text)
- email -> email (text)
- attachment -> attachment (file) (optional)

If your API expects nested fields or JSON in one form field, convert accordingly before sending (e.g., JSON.stringify object into a single form field).

---

## Troubleshooting & tips
- 400/422 responses: check required form fields and field names (match the API docs).
- 401 responses: ensure tokens or auth headers are correct and not expired.
- File upload issues: ensure Content-Type is set by the FormData library (do not set multipart boundary manually).
- Large files: consider streaming or chunked uploads if the API supports them.

---

## Contributing
- Fork the repo, add enhancements (more example scripts, a Postman collection export), and open a PR.
- Add clear examples for other languages or frameworks if you need them.

---

## License
Add your preferred license here (MIT, Apache-2.0, etc.). Example:
MIT © Your Name

---

If you want, I can:
- Generate a sample Postman collection JSON you can import directly,
- Add CI checks or a GitHub Action to validate the spreadsheet format,
- Produce a minimal runnable repository structure with package.json or requirements.txt and sample data.xlsx.

GitHub Copilot Chat Assistant

## @haris-bbil

Generate the format so that it can be added on the md type file

## Copilot

# Update Form-Data — Read Spreadsheet and Submit via POSTMAN

A small example project demonstrating how to read rows from a spreadsheet (CSV or XLSX) and use those rows to build multipart/form-data requests that can be sent from Postman to update an API. Includes sample scripts (Node.js and Python), mapping guidance, and Postman workflow notes to help automate spreadsheet-driven API updates.

---

## Contents
- [Description](#description)  
- [Features](#features)  
- [Prerequisites](#prerequisites)  
- [Quickstart — Node.js](#quickstart---nodejs)  
- [Quickstart — Python](#quickstart---python)  
- [Postman usage](#postman-usage)  
- [Example mapping](#example-mapping)  
- [Troubleshooting & tips](#troubleshooting--tips)  
- [Contributing](#contributing)  
- [License](#license)

---

## Description
This repo shows how to:
- extract rows from a spreadsheet (CSV or XLSX),
- map spreadsheet columns to form-data fields,
- construct multipart/form-data requests (including files/binaries),
- and send those requests via Postman or programmatically to update a target API endpoint.

Intended as a reference for backend developers, QA engineers, and anyone automating API calls using spreadsheet data.

---

## Features
- Node.js and Python example scripts to read spreadsheets and prepare form-data payloads
- Guidance for building Postman requests and using the Postman Runner with data files
- File upload handling (local file paths -> multipart file fields)
- Minimal, easy-to-adapt code to match your API schema

---

## Prerequisites
- Node.js (>= 14) and npm, or Python (>= 3.8) and pip  
- Postman (for running collections)  
- Access to the target API endpoint and any required credentials (API key / bearer token)  
- Spreadsheet file (CSV or XLSX)

---

## Quickstart — Node.js

1. Initialize and install:
```bash
npm init -y
npm install axios form-data xlsx dotenv
```

2. Create a `.env`:
```
API_URL=https://api.example.com/update
API_TOKEN=your_api_token_here
```

3. Example script `index.js`:
```js
require('dotenv').config();
const fs = require('fs');
const path = require('path');
const axios = require('axios');
const FormData = require('form-data');
const XLSX = require('xlsx');

const workbook = XLSX.readFile('data.xlsx'); // or use CSV parser for .csv
const sheet = workbook.Sheets[workbook.SheetNames[0]];
const rows = XLSX.utils.sheet_to_json(sheet);

async function sendRowAsForm(row) {
  const form = new FormData();
  // Map spreadsheet columns to form fields
  form.append('id', row.id);
  form.append('name', row.name);
  form.append('email', row.email);

  // Optional file upload from path stored in spreadsheet
  if (row.attachment && fs.existsSync(row.attachment)) {
    form.append('attachment', fs.createReadStream(path.resolve(row.attachment)));
  }

  const headers = {
    ...form.getHeaders(),
    Authorization: `Bearer ${process.env.API_TOKEN}`,
  };

  const res = await axios.post(process.env.API_URL, form, { headers });
  return res.data;
}

(async () => {
  for (const row of rows) {
    try {
      const result = await sendRowAsForm(row);
      console.log('Updated:', row.id, result);
    } catch (err) {
      console.error('Error updating', row.id, err.response?.data || err.message);
    }
  }
})();
```

4. Run:
```bash
node index.js
```

---

## Quickstart — Python

1. Install dependencies:
```bash
pip install requests openpyxl python-dotenv
```

2. Create a `.env`:
```
API_URL=https://api.example.com/update
API_TOKEN=your_api_token_here
```

3. Example script `update_from_xlsx.py`:
```py
import os
import requests
from openpyxl import load_workbook
from dotenv import load_dotenv

load_dotenv()
API_URL = os.getenv('API_URL')
API_TOKEN = os.getenv('API_TOKEN')

wb = load_workbook('data.xlsx')
ws = wb.active

# header row in the first row
headers = [cell.value for cell in ws[1]]

for row in ws.iter_rows(min_row=2, values_only=True):
    data = dict(zip(headers, row))
    files = {}
    # If spreadsheet stores local file paths in column 'attachment'
    if data.get('attachment'):
        path = data['attachment']
        if os.path.exists(path):
            files['attachment'] = open(path, 'rb')

    payload = {
        'id': data.get('id'),
        'name': data.get('name'),
        'email': data.get('email'),
    }

    headers_req = {'Authorization': f'Bearer {API_TOKEN}'}
    try:
        resp = requests.post(API_URL, data=payload, files=files or None, headers=headers_req)
        resp.raise_for_status()
        print('Updated', data.get('id'), resp.json())
    except Exception as e:
        print('Error updating', data.get('id'), str(e))
    finally:
        if files:
            files['attachment'].close()
```

4. Run:
```bash
python update_from_xlsx.py
```

---

## Postman usage

1. Prepare request:
   - Method: POST (or API-specific method)  
   - URL: target endpoint (use environment variable like `{{API_URL}}`)  
   - Authorization: Bearer Token or other (store token in env var `{{API_TOKEN}}`)  

2. Body tab:
   - Choose `form-data`
   - Add key-value pairs that match API fields (e.g., `id`, `name`, `email`)
   - For files, select type `File` and attach a file

3. Automating with Runner:
   - Use Postman Runner → Import data file (CSV/JSON) to run a collection with many rows
   - Use `{{columnName}}` variables in request body or pre-request script to map data file columns

Notes:
- Postman Runner does not automatically upload local files referenced in CSV rows for every environment; file uploads may require manual selection or Postman file variables. Consult Postman docs if you rely on automated file attachments.
- Consider using programmatic scripts (Node/Python above) when you need to upload local files programmatically with each row.

---

## Example mapping
Spreadsheet columns: `id`, `name`, `email`, `attachment`  
Form-data mapping:
- `id` -> id (text)
- `name` -> name (text)
- `email` -> email (text)
- `attachment` -> attachment (file) (optional)

If the API expects nested JSON in a form field, JSON.stringify the object into a single form field value (or use a separate JSON body depending on the API).

---

## Troubleshooting & tips
- 400 / 422: verify required fields and exact field names expected by the API.  
- 401: check tokens/credentials; confirm token scope and expiration.  
- File upload issues: let FormData set the Content-Type with boundary; do not set multipart boundary manually.  
- Large files: use streaming/chunked uploads if supported by the API.  
- Debugging: log the built form-data headers and small test payloads before bulk runs.

---

## Contributing
- Fork the repository, add examples (additional languages, improved Postman collection, CI checks), then open a PR.  
- Add tests or example data (e.g., `data.xlsx`) to demonstrate mapping.  
- Document any new scripts and update README examples.

---

## License
Add your preferred license here (example below). Replace with your chosen license file if present.

MIT © Your Name

---

If you want, I can also:
- generate a sample Postman collection JSON you can import directly,
- add a minimal runnable repository structure (package.json / requirements.txt) and sample data.xlsx,
- or produce a GitHub Action to validate spreadsheet format.
