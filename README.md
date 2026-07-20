[skill_ocrai.md](https://github.com/user-attachments/files/30193766/skill_ocrai.md)
---
name: skill-ocrai
description: OCR + AI extraction for single-file HTML tools that process clean printed screenshots or scans. Use when building or updating HTML apps that need local OCR with Tesseract.js, then structured extraction with OpenRouter text models such as nvidia/nemotron-3-ultra-550b-a55b:free, plus localStorage saving, preview editing, and JSON import/export.
---

# OCR + AI Extractor

Use this skill for self-contained HTML apps that turn pasted or uploaded screenshots into structured records.

## Recommended flow

1. Accept image input from file picker and clipboard paste.
2. Compress the image before storing it.
3. Run OCR locally with Tesseract.js.
4. Send OCR text to OpenRouter for structured JSON extraction.
5. Show editable preview fields.
6. Save records to localStorage.
7. Support history, view, delete, export, and import.

## When to use

- Clean printed text in bills, receipts, invoices, forms, and screenshots.
- Single-file HTML deliverables.
- Users bring their own OpenRouter API key.
- You want to avoid sending raw images to a vision model.

## When not to use

- Handwriting.
- Very blurry or distorted images.
- Real-time extraction under 1 second.
- Tasks that need a paid vision model instead of OCR.

## Core pattern

Use Tesseract.js as the OCR layer and OpenRouter as the text-extraction layer.

```javascript
async function extractTextFromImage(image) {
  const result = await Tesseract.recognize(image, "eng", {
    logger: (m) => console.log(m)
  });
  return result.data.text;
}

async function extractStructuredData(rawText, apiKey) {
  const response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${apiKey}`
    },
    body: JSON.stringify({
      model: "nvidia/nemotron-3-ultra-550b-a55b:free,
      messages: [
        {
          role: "system",
          content: "Extract structured fields from OCR text and return ONLY valid JSON."
        },
        {
          role: "user",
          content: `OCR text: ${rawText}`
        }
      ]
    })
  });

  const data = await response.json();
  return JSON.parse(data.choices[0].message.content);
}
```

## Image compression

Compress images before saving them to `localStorage`.

```javascript
function compressImage(file) {
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onload = (e) => {
      const img = new Image();
      img.onload = () => {
        const canvas = document.createElement("canvas");
        const maxWidth = 800;
        const scale = Math.min(1, maxWidth / img.width);
        canvas.width = Math.round(img.width * scale);
        canvas.height = Math.round(img.height * scale);
        const ctx = canvas.getContext("2d");
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        resolve(canvas.toDataURL("image/jpeg", 0.7));
      };
      img.src = e.target.result;
    };
    reader.readAsDataURL(file);
  });
}
```

## Prompting

Keep the system prompt strict and JSON-only.

```text
You are a bill extractor. Return ONLY valid JSON.
If a field is missing, use null.
Use normalized dates in YYYY-MM-DD when possible.
```

## Troubleshooting

- If OCR text is garbled, check image quality and orientation.
- If OpenRouter returns invalid JSON, tighten the system prompt and truncate overly long OCR text.
- If extraction feels slow, reduce image size before OCR.

## Output expectations

Prefer a single HTML file with:

- image input
- OCR button
- editable preview
- save to localStorage
- history list
- screenshot viewer
- JSON import/export
