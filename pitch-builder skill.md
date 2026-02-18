---
name: pitch-builder
description: >
  Use this skill when the user has a business report and wants to turn it into a client pitch.
  Produces three outputs: a PPTX slide deck, a PDF leave-behind, and a spoken script.
  Before building anything, asks questions to calibrate tone for the specific client, then
  shows the user a list of recommendations from the report and lets them choose which ones
  to include in the pitch.
---

# Pitch Builder

Transforms a business report into a polished, client-ready pitch. Produces three outputs:
a slide deck (.pptx), a PDF leave-behind, and a spoken script you can read from in the meeting.

This is a formal, professional tool — but the tone adapts to the specific client based on
answers you provide. Every pitch is built around the strategies YOU choose to include,
not the whole report.

---

## Step 1: Load the Report

Ask the user to share the business report. They can:
- Upload the .docx or PDF file
- Paste the content directly

Once loaded, extract and internally list every recommendation or strategy in the report,
numbered clearly. Also note:
- Company name and industry
- Key problems identified
- Budget range (if available)
- Timeline (if available)

---

## Step 2: Calibrate the Tone

Before selecting strategies, ask the user these questions to understand the client. Ask all
at once in a clean numbered list — do not ask one by one:

1. **What kind of company is this client?** (small family business, mid-size company, corporation?)
2. **Who will be in the room?** (owner/founder, executives, a finance team, mixed?)
3. **How tech-savvy is this client?** (very comfortable with technology / somewhat / not at all)
4. **What's your relationship with them?** (first meeting / already know them / long-term contact)
5. **What's their biggest concern likely to be?** (cost, complexity, risk, time, staff buy-in?)

Based on the answers, set one of these tone profiles for the pitch:

| Profile | When to use | Style |
|---------|-------------|-------|
| **Executive** | C-suite, first meeting, formal | Concise, ROI-focused, minimal tech, big numbers |
| **Partner** | Owner/founder, existing relationship | Direct, honest, conversational but professional |
| **Technical** | Tech-savvy audience, mixed room | More detail, specific tools named, implementation steps |
| **Cautious** | Risk-averse, not tech-savvy, skeptical | Extra simple, heavy on proof/examples, small steps framed as low-risk |

Confirm the tone with the user before moving on:
> "Based on what you told me, I'd pitch this as [Profile] — direct and ROI-focused, light on
> technical detail. Does that feel right?"

---

## Step 3: Show the Strategy List

Display all recommendations from the report as a numbered list. Format it clearly:

---
Here are all the strategies from the report. Pick the ones you want to include in the pitch —
you can choose as many or as few as you want:

1. [Strategy name] — [one sentence description]
2. [Strategy name] — [one sentence description]
3. [Strategy name] — [one sentence description]
...

Which ones do you want to pitch?

---

Wait for the user's selection before building anything. They may say "1, 3, and 5" or
"just the first three" or "all of them except #4."

Once confirmed, proceed to build the three outputs.

---

## Step 4: Build the Three Outputs

Build all three outputs based on the selected strategies and tone profile.

---

### Output A: Slide Deck (.pptx)

Use `pptxgenjs` to create the deck. Install with: `npm install -g pptxgenjs`

#### Slide Structure

| Slide | Title | Content |
|-------|-------|---------|
| 1 | Cover | Client company name, "AI Transformation Proposal", your name, date |
| 2 | The Situation Today | 3–4 pain points from the report — specific, not generic |
| 3 | The Opportunity | What's possible in 12 months with the right tools — 2–3 big outcomes |
| 4–N | One slide per selected strategy | Strategy name, the problem it solves, how it works (simple), expected impact |
| N+1 | Investment & ROI | Cost range, timeline, key ROI metric per strategy |
| N+2 | Next Steps | 3 concrete actions to move forward — specific, not vague |
| Last | Thank You / Contact | Clean close slide |

#### Design Rules

- **Color palette**: Use "Midnight Executive" (navy `1E2761`, ice blue `CADCFC`, white) for
  formal clients. Use "Teal Trust" (`028090`, `00A896`, `02C39A`) for tech-forward clients.
  Use "Warm Terracotta" (`B85042`, `E7E8D1`, `A7BEAE`) for family business / warm relationship.
  Match the palette to the tone profile.
- **Dark cover and close slides**, light content slides
- **Every slide needs a visual element** — icon, large stat callout, or shape block
- **No bullet-only slides** — use icon + text rows, two-column layouts, or stat callouts
- **Font**: Trebuchet MS for headers (36–44pt bold), Calibri for body (14–16pt)
- **Large numbers**: If a strategy has a strong metric (e.g., "save 3hrs/day"), make it huge
  — 60–72pt — with a small label below. These are the most memorable slides.
- **NEVER use accent lines under titles**
- **Keep text minimal** — slides are visual support, not a document

#### pptxgenjs Code Pattern

```javascript
const pptxgen = require('pptxgenjs');
const prs = new pptxgen();

prs.layout = 'LAYOUT_WIDE'; // 13.33 x 7.5 inches

// Cover slide example
let slide = prs.addSlide();
slide.background = { color: '1E2761' };
slide.addText('AI TRANSFORMATION PROPOSAL', {
  x: 0.5, y: 2.0, w: 12.33, h: 1.2,
  fontSize: 40, bold: true, color: 'FFFFFF',
  fontFace: 'Trebuchet MS', align: 'center'
});
slide.addText('Client Name — Industry', {
  x: 0.5, y: 3.4, w: 12.33, h: 0.6,
  fontSize: 22, color: 'CADCFC',
  fontFace: 'Calibri', align: 'center'
});

await prs.writeFile({ fileName: '/home/claude/pitch.pptx' });
```

#### QA the Deck

After generating, convert to images and visually inspect:
```bash
python scripts/office/soffice.py --headless --convert-to pdf pitch.pptx
pdftoppm -jpeg -r 150 pitch.pdf slide
```

Check every slide for: text overflow, overlapping elements, low contrast, empty slides,
leftover placeholder text. Fix before delivering.

---

### Output B: PDF Leave-Behind

A clean, printable one or two-page document the client can take away from the meeting.
More detailed than the slides — this is what they read after you leave.

Use `reportlab` to generate it. Structure:

**Page 1:**
- Header: Client name + "AI Transformation Proposal" + date
- Section: "Where You Are Today" — 3–4 bullet pain points
- Section: "What We're Proposing" — selected strategies listed with one paragraph each:
  what it is, how it works simply, expected impact

**Page 2 (if needed):**
- Section: "Investment & Timeline" — cost range per strategy, implementation timeline
- Section: "Why Now" — 2–3 sentences on the cost of waiting / opportunity cost
- Section: "Proposed Next Steps" — 3 numbered actions
- Footer: Your contact info

#### reportlab Pattern

```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, HRFlowable
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib import colors
from reportlab.lib.units import inch

doc = SimpleDocTemplate(
    '/home/claude/pitch_leave_behind.pdf',
    pagesize=letter,
    rightMargin=inch, leftMargin=inch,
    topMargin=inch, bottomMargin=inch
)

styles = getSampleStyleSheet()
# Customize styles for professional look
title_style = ParagraphStyle('CustomTitle',
    parent=styles['Title'],
    fontSize=22, textColor=colors.HexColor('#1E2761'),
    spaceAfter=6
)
heading_style = ParagraphStyle('CustomHeading',
    parent=styles['Heading2'],
    fontSize=13, textColor=colors.HexColor('#028090'),
    spaceBefore=16, spaceAfter=6
)
body_style = ParagraphStyle('CustomBody',
    parent=styles['Normal'],
    fontSize=10.5, leading=16,
    spaceAfter=8
)

story = []
story.append(Paragraph("AI Transformation Proposal", title_style))
story.append(Paragraph("Client Name  |  Date", styles['Normal']))
story.append(HRFlowable(width="100%", thickness=1, color=colors.HexColor('#1E2761')))
story.append(Spacer(1, 12))
# ... add sections
doc.build(story)
```

---

### Output C: Spoken Script

A word-for-word script the user can read from (or use as a guide) during the meeting.

Structure it as a natural conversation, not a formal speech. Written in first person ("I", "we").
Match the tone profile — Executive scripts are tight and punchy. Partner scripts are warmer.
Cautious scripts spend more time on reassurance and small steps.

Format:

```
[COVER SLIDE]
"Good morning / Good afternoon. Thank you for having me.
My name is [your name] and today I want to share something
I genuinely think can change how this business operates day-to-day..."

[SITUATION SLIDE]
"Before I get into solutions, I want to make sure I've understood
your reality correctly. From what I've seen, the three biggest
friction points right now are..."

[STRATEGY SLIDE: WhatsApp Chatbot]
"The first thing I'd recommend is..."
"What this looks like in practice is..."
"For a business like yours, the impact would be..."

[INVESTMENT SLIDE]
"In terms of investment..."
"The way I think about ROI here is..."

[NEXT STEPS]
"Here's what I'd suggest as the next three steps..."

[CLOSE]
"I'm confident this is the right direction. The question
isn't whether this technology works — it does. The question
is whether we start now or six months from now..."
```

Deliver the script as a plain text or markdown file. Keep it natural — not too long,
not too scripted. A good pitch meeting is 20–30 minutes, so the script should reflect that.

---

## Step 5: Deliver All Three Outputs

Once all three files are generated:
1. Copy them to `/mnt/user-data/outputs/`
2. Present them to the user with `present_files`
3. Give a brief summary of what's in each file

Then ask:
> "Want to do a quick run-through of the script together, or adjust anything in the pitch
> before your meeting?"

---

## Tone Profiles in Practice

### Executive Tone
- Lead with numbers and outcomes, not process
- Keep each strategy to one slide, one metric, one action
- Avoid tool names — say "an automated system" not "a WATI chatbot"
- Script is tight: 2–3 sentences per slide max

### Partner Tone
- More conversational, use their company name often
- Can go deeper on the "how" since they trust you
- Script can include light personal observations ("what I noticed when I looked at your data...")
- Still professional, but warmer

### Technical Tone
- Name tools, show timelines, include architecture basics
- Strategy slides can have a simple "how it works" diagram (shape-based in pptx)
- Script can handle questions — add "anticipated Q&A" section at the end

### Cautious Tone
- Frame everything as "low risk, high reward"
- Lead with Phase 1 only — don't overwhelm with the full roadmap
- Use social proof: "businesses similar to yours have seen..."
- Script includes explicit objection-handling moments

---

## Error Handling

- If the user selects only 1–2 strategies, build a tight focused pitch around just those — don't pad it
- If no cost data is in the report, use reasonable ranges and flag them as estimates
- If the report is in Spanish, deliver all three outputs in Spanish
- If the user wants to change their strategy selection after seeing the outline, update before building
