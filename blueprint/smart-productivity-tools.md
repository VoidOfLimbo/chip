# Smart Productivity Tools

## Overview
Smart Productivity Tools is a collection of utility features that augment the core planning tools. The initial focus is on document intelligence: **OCR file parsing** and **image processing**. Additional tools will be added as the product grows. These are premium capabilities primarily targeted at Investor users.

---

## Access by Tier

| Tier | Access |
|---|---|
| Public | None |
| Free | None |
| Investor | **Full access** within their own Server; results are persisted |

Smart Productivity Tools are Server Features. Each tool is enabled per-Server by the Server owner via their Feature subscription. Access to results is scoped to the Server where the tool is enabled.

---

## Tool 1: OCR File Parser

### Purpose
Extract structured text and data from uploaded documents (PDFs, scanned images, Word docs, etc.) so users can digitise and act on physical or image-based records.

### Use Cases
- Scan a paper bill and have it auto-create an Expense record.
- Upload a receipt image and extract amount, merchant, and date.
- Parse a contract PDF and extract key terms.
- Extract line items from an invoice.

### Input Formats
- PDF (text-based and scanned/image-based)
- Images: JPEG, PNG, WEBP, TIFF
- (Future: DOCX, XLSX)

### Output
- Extracted plain text.
- Structured field extraction where possible (amount, date, merchant, etc.) — driven by a parse template or AI model.
- Confidence score per extracted field.
- Raw extracted text always available for manual review.

### Data Model

```
uploads
  id              uuid
  user_id         FK → users
  filename        string
  mime_type       string
  path            string    (storage path, not public URL)
  size            integer   (bytes)
  created_at / updated_at

ocr_results
  id              uuid
  upload_id       FK → uploads
  engine          string    (e.g. "tesseract", "google_vision", "aws_textract")
  raw_text        text
  structured_data json      (key-value pairs extracted)
  confidence      float     (0.0–1.0, nullable)
  status          enum: pending | processing | completed | failed
  processed_at    timestamp nullable
  created_at / updated_at
```

### Processing Flow
1. User uploads a file (chunked upload for large files).
2. File is stored in private storage (not publicly accessible).
3. A queued job (`ProcessOcrJob`) picks up the upload.
4. Job sends the file to the configured OCR engine and writes results to `ocr_results`.
5. User is notified (in-app / email) when processing is complete.
6. User reviews the results and optionally maps extracted data to app records (e.g. create an Expense from extracted amount/date).

### OCR Engine Strategy
- Default: **Tesseract** (open-source, runs inside the container — no external API cost).
- Premium / higher accuracy: **Google Cloud Vision** or **AWS Textract** (per-API-call cost, passed through or absorbed — TBD).
- Engine selected via config; can be swapped per environment.

### Security
- Uploaded files stored in `storage/app/private/` — never publicly accessible.
- Files scoped to the uploading user; accessed only via authenticated, signed temporary URLs.
- File type validation on upload (MIME type + extension whitelist).
- File size limit enforced (TBD, e.g. 20 MB).

---

## Tool 2: Image Processor

### Purpose
Perform common image operations on uploaded images: resize, crop, compress, annotate, and extract metadata. Useful for preparing assets, processing receipts, or normalising images before OCR.

### Use Cases
- Resize and compress a photo before attaching it to a plan or expense.
- Auto-rotate and straighten a scanned document.
- Extract EXIF metadata (date taken, GPS coordinates, device).
- Annotate/highlight regions of an image (future).

### Operations (MVP)
| Operation | Description |
|---|---|
| Resize | Scale to specified dimensions, preserving aspect ratio |
| Compress | Reduce file size with configurable quality |
| Crop | Crop to a bounding box |
| Auto-rotate | Correct orientation based on EXIF data |
| Format convert | Convert between JPEG, PNG, WEBP |
| Metadata extract | Return EXIF/IPTC metadata as JSON |

### Data Model

```
image_processing_jobs
  id              uuid
  upload_id       FK → uploads
  user_id         FK → users
  operations      json      (ordered list of operations with params)
  output_path     string    (storage path of processed result)
  status          enum: pending | processing | completed | failed
  processed_at    timestamp nullable
  created_at / updated_at
```

### Processing Flow
1. User selects an image (existing upload or new upload).
2. User configures desired operations via the UI (or accepts defaults).
3. `ProcessImageJob` queued; applies operations using **Intervention Image** (PHP) or delegates to **Imagick**.
4. Processed output stored in private storage.
5. User downloads or attaches the result.

### Library
- **Intervention Image v3** (Laravel-native, Imagick or GD backend).
- No external API required for basic operations.

---

## Tool 3+: Future Tools (TBD)

Potential additions as the feature set grows:

| Tool | Description |
|---|---|
| Document summariser | AI-generated summary of uploaded documents |
| Expense auto-import | Parse bank statement CSV/PDF into Expense Planner records |
| Smart search | Full-text search across all plans, expenses, and uploaded documents |
| Data export bundle | Export all user data (plans, expenses, uploads) as a ZIP |

---

## Permissions & Abilities

| Ability Slug | Add-on? | Description |
|---|---|---|
| `ocr_parser` | Yes | Unlock full (persistent) OCR File Parser for tenant |
| `image_processor` | Yes | Unlock full (persistent) Image Processor for tenant |
| `productivity_suite` | Yes (bundle) | Unlocks all current Smart Productivity Tools |

All Smart Productivity Tools are **Investor-only** for full use via add-on Abilities. Supporter users get ephemeral demo access without needing an Ability grant.

| Permission | `owner` | `free` | `supporter` | `investor` |
|---|---|---|---|---|
| `tools.ocr` (demo) | ✅ | ❌ | ✅ (ephemeral) | ✅ |
| `tools.ocr` (full/persistent) | ✅ | ❌ | ❌ | ✅ (Ability: `ocr_parser`) |
| `tools.image_processor` (demo) | ✅ | ❌ | ✅ (ephemeral) | ✅ |
| `tools.image_processor` (full/persistent) | ✅ | ❌ | ❌ | ✅ (Ability: `image_processor`) |
| `uploads.create` | ✅ | ❌ | ✅ (ephemeral) | ✅ (tenant) |
| `uploads.read` | ✅ | ❌ | ❌ | ✅ (tenant) |
| `uploads.delete` | ✅ | ❌ | ❌ | ✅ (tenant) |

---

## Infrastructure Notes
- OCR and image processing are CPU-intensive — always run in queued jobs, never synchronously.
- Use a dedicated queue worker (or priority queue) for processing jobs to avoid blocking other queue work.
- In Sail, Tesseract and Imagick/GD are available; ensure they are included in the Sail Docker image configuration.
- Consider storage quotas per user (TBD) to manage disk usage.

---

## Open Questions
- Should Supporter users get any Smart Productivity access (e.g. image processing only)?
- Is Tesseract accuracy sufficient for MVP, or should an external OCR API be used from the start?
- What is the per-user storage quota for uploads?
- Should processed images/OCR results ever auto-expire and be deleted?
- AI-powered document summarisation — is this in scope for the initial Investor add-on set?
- Should OCR results be directly linkable to Expense Planner (auto-create expense from receipt)?
