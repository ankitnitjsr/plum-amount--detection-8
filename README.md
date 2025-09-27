💊 AI-Powered Amount Detection for Medical Bills

An end-to-end demo system to automatically identify financial figures in medical documents (typed or scanned), correct OCR misreads, standardize numeric formats, and label amounts by context (Total, Paid, Due).

The project offers a single API and a simple React demo UI to test the pipeline.

🔍 Core Workflow

OCR – Extract text from uploaded images using tesseract.js (eng default, eng+hin supported).

Numeric Normalization – Fix common OCR mistakes (e.g., O → 0, l/I → 1, S → 5, B → 8), remove currency marks (₹, INR, RS), clean commas/spaces.

Context Classification – Assign each number to a type (total_bill, paid, due) using nearby keywords.

Provenance – Every output number includes a snippet of source text for traceability.

✨ Features

Works with both text and image input (PNG/JPG).

Guardrails against noisy or invalid inputs.

Each pipeline step can be invoked separately for debugging.

Frontend built with React + Vite for quick demo (text/file upload).

Structured JSON outputs with confidence scores.

🏗 Project Layout
/client
  ├─ index.html
  └─ src/
      ├─ main.jsx
      ├─ App.jsx
      └─ components/
          ├─ UploadForm.jsx
          └─ Results.jsx

/server
  └─ src/
      ├─ index.js            # Express server (file upload configured once)
      ├─ routes/
      │   └─ amount.js       # API routes
      ├─ services/
      │   ├─ pipeline.js     # Stepwise + full pipeline
      │   ├─ ocr.js          # OCR handler
      │   ├─ extract.js      # Token extraction
      │   ├─ normalize.js    # OCR error correction
      │   └─ classify.js     # Context classifier
      ├─ utils/
      │   └─ schema.js
      └─ schemas/            # JSON schemas for validation

🚀 Setup Guide
Prerequisites

Node.js ≥ 18

npm ≥ 9

1. Install Dependencies
cd server && npm install
cd ../client && npm install

2. Configure Server (server/src/index.js)

Make sure file upload is mounted once:

import express from "express";
import cors from "cors";
import fileUpload from "express-fileupload";
import dotenv from "dotenv";
import amountRouter from "./routes/amount.js";

dotenv.config();
const app = express();

app.use(cors());
app.use(express.json({ limit: "2mb" }));
app.use(express.urlencoded({ extended: true }));

// mount fileUpload once
app.use(fileUpload({ useTempFiles: false, createParentPath: true }));

app.get("/", (_req, res) =>
  res.json({ ok: true, service: "AI Amount Detection" })
);

app.use("/api/v1", amountRouter);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () =>
  console.log(`Server running → http://localhost:${PORT}`)
);


⚠️ Error "Unexpected end of form" means fileUpload was mounted twice.

3. Run Development Servers
# backend
cd server
npm run dev   # → http://localhost:5000

# frontend
cd client
npm run dev   # → http://localhost:5173


(Optional) Add a proxy in client/vite.config.js:

server: {
  proxy: {
    "/api": {
      target: "http://localhost:5000",
      changeOrigin: true
    }
  }
}

🌐 API Overview

Base URL: http://localhost:5000/api/v1

Health Check

GET / → { ok: true, service: "AI Amount Detection" }

Step 1 — Extract

POST /extract

Text Input:

{ "text": "Total: INR 1200 | Paid: 1000 | Due: 200" }


Image Input: multipart/form-data with key = file

✅ Example Response:

{
  "status": "ok",
  "cleanedText": "Total: INR 1200 Paid: 1000 Due: 200",
  "raw_tokens": ["1200","1000","200"],
  "currency_hint": "INR",
  "confidence": 0.74
}

Step 2 — Normalize

POST /normalize

{ "tokens": ["l200","1000","200"] }


✅ Example Response:

{
  "normalized_amounts": [1200, 1000, 200],
  "normalization_confidence": 0.82
}

Step 3 — Classify

POST /classify

{
  "text": "Total: INR 1200 | Paid: 1000 | Due: 200",
  "numbers": [1200, 1000, 200]
}


✅ Example Response:

{
  "amounts": [
    { "type": "total_bill", "value": 1200, "source": "Total: INR 1200" },
    { "type": "paid",       "value": 1000, "source": "Paid: 1000" },
    { "type": "due",        "value": 200,  "source": "Due: 200" }
  ],
  "confidence": 0.80
}

Step 4 — Full Pipeline

POST /process

Supports text JSON or file upload.

✅ Example Response:

{
  "currency": "INR",
  "amounts": [
    { "type": "total_bill", "value": 1200, "source": "Total: INR 1200" },
    { "type": "paid", "value": 1000, "source": "Paid: 1000" },
    { "type": "due", "value": 200, "source": "Due: 200" }
  ],
  "status": "ok"
}

🧪 Testing

Use cURL:

curl -X POST http://localhost:5000/api/v1/process \
  -H "Content-Type: application/json" \
  -d '{"text":"Total: INR 1200 | Paid: 1000 | Due: 200"}'


Or set up a Postman collection with routes:

Extract (Text / Image)

Normalize (JSON / Image)

Classify

Process (Text / Image)

🖥 React Frontend

In UploadForm.jsx:

If file uploaded → FormData with key = file

If only text → JSON { text }

❌ Don’t stringify FormData.
❌ Don’t rename the file key.
❌ Don’t set Content-Type manually when using FormData.

⚙️ Extensions & Next Steps

Multi-language OCR (eng+hin) for bilingual receipts.

Confidence scoring tuned by organization keywords.

DB persistence for audits.

File size/type validation + rate limiting for production security.

📜 Scripts

Backend

npm run dev    # dev with nodemon
npm start      # production


Frontend

npm run dev
npm run build
