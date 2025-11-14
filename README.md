<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>txt ⇄ PDF — Mobile Safe</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #f8fafc;
      padding: 16px;
      color: #1e293b;
    }
    .container {
      max-width: 768px;
      margin: 0 auto;
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 16px rgba(0,0,0,0.05);
      padding: 20px;
    }
    h1 {
      text-align: center;
      margin-bottom: 16px;
      color: #4f46e5;
      font-weight: 700;
      font-size: 1.5rem;
    }
    h2 {
      margin: 20px 0 8px;
      font-size: 1.1rem;
      color: #334155;
    }
    textarea {
      width: 100%;
      min-height: 120px;
      padding: 12px;
      border: 1px solid #cbd5e1;
      border-radius: 8px;
      font-family: ui-monospace, monospace;
      font-size: 14px;
      resize: vertical;
      margin-bottom: 12px;
    }
    .btn {
      background: #4f46e5;
      color: white;
      border: none;
      padding: 10px 16px;
      border-radius: 8px;
      font-weight: 600;
      cursor: pointer;
      transition: background 0.2s;
      margin-right: 8px;
      margin-top: 4px;
      font-size: 14px;
    }
    .btn:hover { background: #4338ca; }
    .btn-outline {
      background: transparent;
      border: 1px solid #cbd5e1;
      color: #475569;
    }
    .btn-outline:hover {
      background: #f1f5f9;
    }
    #pdfInput {
      display: none;
    }
    .status {
      margin-top: 8px;
      font-size: 0.875rem;
      color: #64748b;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>📄 txt ⇄ PDF — Mobile Safe</h1>

    <!-- Text to PDF -->
    <h2>📝 Text to PDF</h2>
    <textarea id="textInput" placeholder="Paste or type text here..."></textarea>
    <button class="btn" onclick="textToPDF()">Download PDF</button>
    <button class="btn btn-outline" onclick="loadSavedText()">Load Saved</button>
    <button class="btn btn-outline" onclick="saveTextLocally()">Save Text</button>

    <!-- PDF to Text -->
    <h2>🔍 PDF to Text</h2>
    <button class="btn" onclick="document.getElementById('pdfInput').click()">
      Upload PDF
    </button>
    <input type="file" id="pdfInput" accept=".pdf" onchange="pdfToText(event)" />
    <div class="status" id="pdfStatus"></div>
    <textarea id="textOutput" placeholder="Extracted text appears here..." readonly></textarea>
  </div>

  <script>
    // Auto-load saved text on start
    window.onload = () => {
      const saved = localStorage.getItem('userText');
      if (saved) document.getElementById('textInput').value = saved;
    };

    // Save text to localStorage
    function saveTextLocally() {
      const text = document.getElementById('textInput').value;
      localStorage.setItem('userText', text);
      alert('✅ Text saved locally!');
    }

    // Load saved text
    function loadSavedText() {
      const saved = localStorage.getItem('userText');
      if (saved) {
        document.getElementById('textInput').value = saved;
      } else {
        alert('ℹ️ No saved text found.');
      }
    }

    // Text to PDF — optimized for mobile
    function textToPDF() {
      const text = document.getElementById('textInput').value.trim();
      if (!text) {
        alert('⚠️ Please enter text first.');
        return;
      }

      try {
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF({
          format: 'a4',
          orientation: 'portrait'
        });

        const lines = doc.splitTextToSize(text, 180); // Limit line width
        let y = 20;

        for (let i = 0; i < lines.length; i++) {
          if (y > 270) { // Page height limit
            doc.addPage();
            y = 20;
          }
          doc.text(lines[i], 20, y);
          y += 8; // Line spacing
        }

        doc.save('converted.pdf');

      } catch (err) {
        alert('❌ PDF generation failed. Try shorter text.');
        console.error(err);
      }
    }

    // Simple PDF to Text (no heavy processing)
    async function pdfToText(event) {
      const file = event.target.files[0];
      if (!file) return;

      const statusEl = document.getElementById('pdfStatus');
      statusEl.textContent = '🔄 Processing...';

      try {
        const arrayBuffer = await file.arrayBuffer();
        const pdf = await pdfjsLib.getDocument({  arrayBuffer }).promise;
        let fullText = '';

        for (let i = 1; i <= pdf.numPages && i <= 5; i++) { // Limit to 5 pages for mobile
          const page = await pdf.getPage(i);
          const content = await page.getTextContent();
          const pageText = content.items.map(item => item.str).join(' ');
          fullText += pageText + '\n\n';
        }

        document.getElementById('textOutput').value = fullText.trim();
        statusEl.textContent = '✅ Done!';
      } catch (err) {
        console.error(err);
        statusEl.textContent = '❌ Failed (scanned/encrypted)';
        document.getElementById('textOutput').value = '';
      }
    }
  </script>
</body>
</html>
