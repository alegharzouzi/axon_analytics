---
name: business-report
description: >
  Use this skill when the user uploads any kind of data files (CSV, Excel, PDF, Word, HTML, images)
  and wants a business report generated from them. The report analyzes the data, extracts KPIs,
  identifies trends, and provides actionable recommendations. Output format is chosen based on
  the complexity and type of data.
---

# Business Report Generator

Transforms raw data files into clear, actionable business reports. Designed for personal use —
the goal is to quickly understand what the data says and what to do about it.

---

## Step 1: Understand the Context (Ask First)

Before analyzing anything, ask the user these clarifying questions (all at once, not one by one):

1. **What is this data about?** (e.g., "monthly sales from a client", "survey results from a vendor", "financial statements")
2. **What decision or action are you trying to make?** (e.g., "decide whether to work with them", "understand their performance", "spot problems")
3. **Is there anything specific you want to focus on or are concerned about?** (optional — skip if they say nothing)

Keep the questions short and conversational. Don't ask for information you can already infer from the filenames or content.

---

## Step 2: Analyze the Files

Read and process all uploaded files. Handle each type appropriately:

| File Type | How to Read |
|-----------|-------------|
| CSV / Excel (.xlsx) | Load with pandas, inspect shape, dtypes, nulls, summary stats |
| PDF | Extract text with pdfplumber or PyPDF2; look for tables, figures, key numbers |
| Word (.docx) | Extract text with python-docx or pandoc |
| HTML | Parse with BeautifulSoup; extract main content, tables, lists |
| Images (.png, .jpg) | Describe visible charts, graphs, data; extract numbers where readable |
| Plain text (.txt, .md) | Read directly |

**For each file, identify:**
- What kind of data it contains
- Time range (if applicable)
- Key numeric columns or metrics
- Any obvious data quality issues (missing values, outliers, inconsistencies)

---

## Step 3: Extract Insights

Analyze the data to find:

### KPIs & Key Metrics
- Pull the most important numbers (totals, averages, rates, counts)
- Compare against benchmarks if available (prior period, industry standard, targets)
- Flag anything that looks unusually high, low, or unexpected

### Trends & Patterns
- Look for changes over time (growth, decline, seasonality)
- Identify correlations between variables
- Note any anomalies or outliers

### Risks & Concerns
- Highlight anything that could be a problem (declining metrics, data inconsistencies, missing information, red flags)

### Opportunities & Positives
- What's working well?
- What stands out as a strength?

---

## Step 4: Choose the Output Format

Pick the best format based on the data:

| Situation | Output Format |
|-----------|---------------|
| Rich data with multiple sections, tables, charts | `.docx` (Word document) |
| Data-heavy with lots of numbers and comparisons | `.docx` or HTML |
| Simple summary, lightweight data | Markdown in chat or `.md` file |
| Visual/chart-heavy content | HTML with embedded charts |

**Default to `.docx`** when unsure — it's the most versatile for review and sharing.

---

## Step 5: Generate the Report

Structure the report with these sections (skip any that aren't relevant):

### 1. Executive Summary
2–4 sentences. What is this data? What's the most important takeaway? What should be done?

### 2. Data Overview
Brief description of the files analyzed: what they contain, time range, scope.

### 3. Key Metrics & KPIs
Table or list of the most important numbers. Include comparisons where possible.

### 4. Trends & Patterns
What's changing over time? What patterns stand out?

### 5. Risks & Red Flags
Anything concerning that needs attention.

### 6. Action Items
Concrete, specific things to do. Numbered list. Start each with a verb.
Example: "1. Follow up with the vendor about the 23% drop in Q3 delivery rate."

### 7. Appendix (optional)
Raw tables, detailed breakdowns, or data notes that don't fit in the main report.

---

## Creating the .docx Report

When generating a Word document, use the `docx` npm package:

```bash
npm install -g docx
```

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
        HeadingLevel, AlignmentType, BorderStyle, WidthType, ShadingType } = require('docx');
const fs = require('fs');

// US Letter, 1-inch margins
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } },
    paragraphStyles: [
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 32, bold: true, font: "Arial", color: "1F3864" },
        paragraph: { spacing: { before: 240, after: 120 }, outlineLevel: 0 } },
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 26, bold: true, font: "Arial", color: "2E5F8A" },
        paragraph: { spacing: { before: 200, after: 80 }, outlineLevel: 1 } },
    ]
  },
  sections: [{
    properties: {
      page: {
        size: { width: 12240, height: 15840 },
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }
      }
    },
    children: [/* report content */]
  }]
});

Packer.toBuffer(doc).then(buf => fs.writeFileSync('report.docx', buf));
```

After creating, validate:
```bash
python scripts/office/validate.py report.docx
```

Then copy to `/mnt/user-data/outputs/report.docx` and present to the user.

---

## Tone & Style

- **Write for yourself** — direct, no fluff, no corporate filler
- **Lead with conclusions** — don't bury the key finding at the end
- **Be specific** — "Revenue dropped 18% in October" not "there was a decline"
- **Flag uncertainty** — if the data is unclear or incomplete, say so
- **Keep it scannable** — short paragraphs, clear headings, use tables for numbers

---

## Error Handling

- If a file can't be read, note it and continue with the others
- If data is too sparse to draw conclusions, say so clearly in the report
- If the user's goal doesn't match the available data, flag the gap and explain what you *can* analyze
