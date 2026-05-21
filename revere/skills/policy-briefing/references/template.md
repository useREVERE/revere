# Policy Briefing — docx-js Reference Template

This is the tested code template for generating a policy briefing `.docx`. Copy and adapt it for each briefing. The structural patterns and formatting values are stable.

## Setup and Imports

```javascript
const fs = require("fs");
const {
  Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
  Header, Footer, AlignmentType, LevelFormat, ExternalHyperlink,
  HeadingLevel, BorderStyle, WidthType, ShadingType,
  TabStopType, PageBreak, PageNumber
} = require("docx");
```

## Color Constants

```javascript
const NAVY = "1B3A5C";
const BLUE = "2E75B6";
const LIGHT_BLUE = "D5E8F0";
const DARK_GRAY = "333333";
const MED_GRAY = "666666";
const LIGHT_GRAY = "F2F2F2";
const RED_TAG = "C0392B";   // HIGH priority
const GREEN_TAG = "27AE60"; // LOW priority
const AMBER_TAG = "E67E22"; // MODERATE priority
```

## Shared Layout Constants

```javascript
const border = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const borders = { top: border, bottom: border, left: border, right: border };
const cellMargins = { top: 60, bottom: 60, left: 100, right: 100 };
```

## Helper Functions

These produce every element in the briefing. Use them directly — do not rewrite from scratch.

### Priority Color

```javascript
function priorityColor(p) {
  if (p === "HIGH") return RED_TAG;
  if (p === "MODERATE") return AMBER_TAG;
  return GREEN_TAG;
}
```

### At-a-Glance Table Row

Three-column row: development description, priority tag, recommended action.

```javascript
function atGlanceRow(dev, priority, action) {
  return new TableRow({
    children: [
      new TableCell({
        borders, width: { size: 4600, type: WidthType.DXA },
        margins: cellMargins,
        children: [new Paragraph({ spacing: { before: 40, after: 40 }, children: [
          new TextRun({ text: dev, font: "Arial", size: 18, color: DARK_GRAY })
        ]})]
      }),
      new TableCell({
        borders, width: { size: 1200, type: WidthType.DXA },
        margins: cellMargins,
        verticalAlign: "center",
        children: [new Paragraph({ alignment: AlignmentType.CENTER, spacing: { before: 40, after: 40 }, children: [
          new TextRun({ text: priority, font: "Arial", size: 18, bold: true, color: priorityColor(priority) })
        ]})]
      }),
      new TableCell({
        borders, width: { size: 3560, type: WidthType.DXA },
        margins: cellMargins,
        children: [new Paragraph({ spacing: { before: 40, after: 40 }, children: [
          new TextRun({ text: action, font: "Arial", size: 18, color: DARK_GRAY })
        ]})]
      }),
    ]
  });
}
```

### Section Heading (with blue underline)

```javascript
function sectionHeading(text) {
  return new Paragraph({
    spacing: { before: 360, after: 180 },
    border: { bottom: { style: BorderStyle.SINGLE, size: 4, color: BLUE, space: 4 } },
    children: [new TextRun({ text: text, font: "Arial", size: 26, bold: true, color: NAVY })]
  });
}
```

### Subheading with Priority Tag

```javascript
function subHeading(title, priority) {
  const runs = [new TextRun({ text: title, font: "Arial", size: 22, bold: true, color: NAVY })];
  if (priority) {
    runs.push(new TextRun({ text: "  " }));
    runs.push(new TextRun({ text: priority, font: "Arial", size: 18, bold: true, color: priorityColor(priority) }));
  }
  return new Paragraph({ spacing: { before: 280, after: 100 }, children: runs });
}
```

### Body Text

```javascript
function bodyText(text) {
  return new Paragraph({
    spacing: { before: 60, after: 60 },
    children: [new TextRun({ text, font: "Arial", size: 20, color: DARK_GRAY })]
  });
}
```

### Source Link

Place after the subheading for each item. Uses `ExternalHyperlink` to create a clickable link to the original document.

```javascript
function sourceLink(sourceName, url) {
  return new Paragraph({
    spacing: { before: 20, after: 60 },
    children: [
      new TextRun({ text: "Source: ", font: "Arial", size: 18, color: MED_GRAY }),
      new ExternalHyperlink({
        link: url,
        children: [new TextRun({ text: sourceName, font: "Arial", size: 18, color: BLUE, underline: {} })],
      }),
    ]
  });
}
```

**Usage in context:**
```javascript
subHeading("Treasury AI Innovation Series", "HIGH"),
sourceLink("U.S. Department of the Treasury", "https://home.treasury.gov/news/press-releases/sb0421"),
bodyText("FSOC and Treasury's Artificial Intelligence Transformation Office launched..."),
```

For items with multiple relevant sources, build a single paragraph with multiple hyperlinks separated by " | ":
```javascript
new Paragraph({ spacing: { before: 20, after: 60 }, children: [
  new TextRun({ text: "Sources: ", font: "Arial", size: 18, color: MED_GRAY }),
  new ExternalHyperlink({ link: "https://...", children: [new TextRun({ text: "CoinDesk", font: "Arial", size: 18, color: BLUE, underline: {} })] }),
  new TextRun({ text: " | ", font: "Arial", size: 18, color: MED_GRAY }),
  new ExternalHyperlink({ link: "https://...", children: [new TextRun({ text: "CryptoSlate", font: "Arial", size: 18, color: BLUE, underline: {} })] }),
] })
```

### WHY THIS MATTERS Block (shaded)

```javascript
function whyMatters(text) {
  return new Paragraph({
    spacing: { before: 100, after: 60 },
    shading: { fill: LIGHT_GRAY, type: ShadingType.CLEAR },
    indent: { left: 200, right: 200 },
    children: [
      new TextRun({ text: "WHY THIS MATTERS FOR AIR: ", font: "Arial", size: 20, bold: true, color: NAVY }),
      new TextRun({ text, font: "Arial", size: 20, color: DARK_GRAY })
    ]
  });
}
```

**Note:** Replace "AIR" with the organization's abbreviation. The bold label + regular text pattern is intentional — it lets the reader scan for the analytical content quickly.

### Action Arrow (for Immediate Actions section)

```javascript
function actionArrow(text) {
  return new Paragraph({
    spacing: { before: 40, after: 40 },
    indent: { left: 360 },
    children: [
      new TextRun({ text: "\u2192 ", font: "Arial", size: 20, color: BLUE }),
      new TextRun({ text, font: "Arial", size: 20, color: DARK_GRAY })
    ]
  });
}
```

### Table Header Row (reusable)

```javascript
function tableHeaderRow(labels, widths) {
  return new TableRow({
    children: labels.map((label, i) =>
      new TableCell({
        borders, width: { size: widths[i], type: WidthType.DXA },
        margins: cellMargins,
        shading: { fill: NAVY, type: ShadingType.CLEAR },
        children: [new Paragraph({
          alignment: label === "PRIORITY" ? AlignmentType.CENTER : AlignmentType.LEFT,
          children: [new TextRun({ text: label, font: "Arial", size: 18, bold: true, color: "FFFFFF" })]
        })]
      })
    )
  });
}
```

### Event Row

```javascript
function eventRow(date, lines) {
  return new TableRow({
    children: [
      new TableCell({
        borders, width: { size: 1800, type: WidthType.DXA },
        margins: cellMargins,
        shading: { fill: LIGHT_GRAY, type: ShadingType.CLEAR },
        children: [new Paragraph({ children: [
          new TextRun({ text: date, font: "Arial", size: 18, bold: true, color: DARK_GRAY })
        ] })]
      }),
      new TableCell({
        borders, width: { size: 7560, type: WidthType.DXA },
        margins: cellMargins,
        children: lines.map(l => new Paragraph({
          spacing: { before: 20, after: 20 },
          children: [new TextRun({ text: l, font: "Arial", size: 18, color: DARK_GRAY })]
        }))
      }),
    ]
  });
}
```

## Document Skeleton

This is the outer structure. Fill the `children` array with content elements using the helpers above.

```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 20 } } },
  },
  sections: [{
    properties: {
      page: {
        size: { width: 12240, height: 15840 },
        margin: { top: 1080, right: 1080, bottom: 1080, left: 1080 }
      }
    },
    headers: {
      default: new Header({
        children: [
          new Paragraph({
            alignment: AlignmentType.LEFT,
            border: { bottom: { style: BorderStyle.SINGLE, size: 4, color: BLUE, space: 4 } },
            children: [
              new TextRun({ text: "ORGANIZATION NAME", font: "Arial", size: 16, bold: true, color: NAVY }),
              new TextRun({ text: " | Federal Policy Briefing | Week of [DATE]", font: "Arial", size: 16, color: MED_GRAY }),
            ]
          })
        ]
      })
    },
    footers: {
      default: new Footer({
        children: [
          new Paragraph({
            alignment: AlignmentType.CENTER,
            border: { top: { style: BorderStyle.SINGLE, size: 2, color: "CCCCCC", space: 4 } },
            children: [
              new TextRun({ text: "Prepared for internal use.", font: "Arial", size: 16, color: MED_GRAY }),
            ]
          })
        ]
      })
    },
    children: [
      // TITLE BLOCK
      // Use Monday of the coverage week for [DATE], not the first item's publication date
      new Paragraph({
        alignment: AlignmentType.LEFT,
        spacing: { after: 40 },
        children: [new TextRun({ text: "Federal Policy Briefing", font: "Arial", size: 40, bold: true, color: NAVY })]
      }),
      new Paragraph({
        spacing: { after: 200 },
        children: [new TextRun({ text: "Week of [DATE]", font: "Arial", size: 28, color: BLUE })]
      }),

      // AT A GLANCE
      sectionHeading("AT A GLANCE"),
      new Table({
        width: { size: 9360, type: WidthType.DXA },
        columnWidths: [4600, 1200, 3560],
        rows: [
          tableHeaderRow(["DEVELOPMENT", "PRIORITY", "AIR ACTION"], [4600, 1200, 3560]),
          // Format: "Title - description sentence." (dash separator, no newlines)
          atGlanceRow("Treasury GENIUS Act NPRM - First rulemaking implementing the GENIUS Act. Proposes principles for state stablecoin regime equivalence. Comments due June 2.", "HIGH", "Submit comment letter on consumer protection baselines"),
          // ... more rows
        ]
      }),

      // IMMEDIATE ACTIONS
      sectionHeading("IMMEDIATE ACTIONS"),
      bodyText("Deadline-driven items - recommended action in the next two weeks."),
      actionArrow("Treasury GENIUS Act NPRM: Submit comment letter by June 2 - particularly around consumer protection baselines, reserve transparency, and state supervisory capacity."),
      // ... more arrows

      // THEMATIC SECTIONS (repeat pattern for each domain)
      sectionHeading("SECTION NAME"),
      subHeading("Item Title", "HIGH"),
      sourceLink("Agency Name", "https://..."),
      bodyText("Analytical body paragraph."),
      whyMatters("Organization-specific relevance analysis."),

      // PAGE BREAK before events
      new Paragraph({ children: [new PageBreak()] }),

      // UPCOMING EVENTS
      sectionHeading("UPCOMING EVENTS"),
      bodyText("Items marked \u2605 are highest relevance to core work."),
      new Table({
        width: { size: 9360, type: WidthType.DXA },
        columnWidths: [1800, 7560],
        rows: [
          tableHeaderRow(["DATE", "EVENT"], [1800, 7560]),
          eventRow("4/15", [
            "\u2605 Event title and description",
            "Time, location, relevance note",
          ]),
          // ... more event rows
        ]
      }),
    ]
  }]
});

// Write to file
Packer.toBuffer(doc).then(buffer => {
  fs.writeFileSync("/home/claude/output.docx", buffer);
  console.log("Document created successfully");
});
```

## Post-Generation Checklist

1. **Validate:** `python /mnt/skills/public/docx/scripts/office/validate.py output.docx`
2. **Copy to outputs:** `cp output.docx /mnt/user-data/outputs/`
3. **Present:** Use `present_files` tool to share with user
