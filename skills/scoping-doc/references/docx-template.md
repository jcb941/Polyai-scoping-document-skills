# Scoping Document docx Template

Working skeleton using the `docx` npm library, matching the visual style used across PolyAI scoping docs (Arial body text, simple bullets, light section headers). Adapt content, keep structure and styling helpers.

## Prerequisites

```bash
npm install docx
```

## Template

```javascript
const fs = require("fs");
const {
  Document, Packer, Paragraph, TextRun, HeadingLevel,
  Table, TableRow, TableCell, WidthType, BorderStyle
} = require("docx");

// ---------- Style helpers ----------
function h1(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_1,
    spacing: { before: 240, after: 120 },
    children: [new TextRun({ text, bold: true, font: "Arial", size: 26 })]
  });
}

function h2(text) {
  return new Paragraph({
    heading: HeadingLevel.HEADING_2,
    spacing: { before: 180, after: 100 },
    children: [new TextRun({ text, bold: true, font: "Arial", size: 23 })]
  });
}

function body(text) {
  return new Paragraph({
    spacing: { before: 40, after: 120 },
    children: [new TextRun({ text, font: "Arial", size: 21 })]
  });
}

function bullet(text) {
  return new Paragraph({
    numbering: { reference: "bullets", level: 0 },
    spacing: { before: 40, after: 40 },
    children: [new TextRun({ text, font: "Arial", size: 20 })]
  });
}

function spacer() {
  return new Paragraph({ text: "", spacing: { after: 60 } });
}

function cell(text, opts = {}) {
  return new TableCell({
    width: { size: opts.width || 33, type: WidthType.PERCENTAGE },
    shading: opts.header ? { fill: "EFEFEF" } : undefined,
    children: [new Paragraph({
      children: [new TextRun({ text, font: "Arial", size: 19, bold: !!opts.header })]
    })]
  });
}

// ---------- Table builders ----------
function openItemsTable(rows) {
  const headerRow = new TableRow({
    children: [
      cell("Item", { header: true, width: 20 }),
      cell("Description / Open Question", { header: true, width: 55 }),
      cell("Owner / Needs Confirmation From", { header: true, width: 25 })
    ]
  });
  const dataRows = rows.map(r => new TableRow({
    children: [cell(r.item, { width: 20 }), cell(r.description, { width: 55 }), cell(r.owner, { width: 25 })]
  }));
  return new Table({
    width: { size: 100, type: WidthType.PERCENTAGE },
    rows: [headerRow, ...dataRows]
  });
}

function rolesTable(rows) {
  const headerRow = new TableRow({
    children: [
      cell("Responsibility", { header: true, width: 40 }),
      cell("PolyAI", { header: true, width: 30 }),
      cell("Client", { header: true, width: 30 })
    ]
  });
  const dataRows = rows.map(r => new TableRow({
    children: [cell(r.responsibility, { width: 40 }), cell(r.polyai, { width: 30 }), cell(r.client, { width: 30 })]
  }));
  return new Table({
    width: { size: 100, type: WidthType.PERCENTAGE },
    rows: [headerRow, ...dataRows]
  });
}

function phasesTable(rows) {
  const headerRow = new TableRow({
    children: [
      cell("Phase", { header: true, width: 15 }),
      cell("Description", { header: true, width: 45 }),
      cell("Key Deliverables", { header: true, width: 40 })
    ]
  });
  const dataRows = rows.map(r => new TableRow({
    children: [cell(r.phase, { width: 15 }), cell(r.description, { width: 45 }), cell(r.deliverables, { width: 40 })]
  }));
  return new Table({
    width: { size: 100, type: WidthType.PERCENTAGE },
    rows: [headerRow, ...dataRows]
  });
}

// ---------- Document ----------
const outputPath = process.argv[2] || `${require("os").homedir()}/Desktop/Client_Scoping.docx`;

const doc = new Document({
  numbering: {
    config: [{
      reference: "bullets",
      levels: [{ level: 0, format: "bullet", text: "•", alignment: "left" }]
    }]
  },
  sections: [{
    properties: {},
    children: [
      // Title
      new Paragraph({
        spacing: { after: 300 },
        children: [new TextRun({ text: "[Client Name] — Solution Scoping Document", bold: true, font: "Arial", size: 32 })]
      }),

      // 1. Background
      h1("1. Background"),
      body("[Client] is a [description]. [Client] uses [telephony platform] as its telephony platform and [booking/CRM engine]. This document covers the scope for a PolyAI deployment, scoped from discovery and working sessions held in [month/year]."),
      spacer(),

      // 2. How It Works Today (new deployment) / Current Deployment (expansion)
      h1("2. How It Works Today"),
      body("[Describe current manual process, pain points, compliance gaps, what's not self-service today.]"),
      spacer(),

      // 3. Objectives
      h1("3. Objectives"),
      bullet("[Outcome-oriented objective 1]"),
      bullet("[Outcome-oriented objective 2]"),
      bullet("[Outcome-oriented objective 3]"),
      spacer(),

      // 4. Use Cases
      h1("4. Use Cases"),
      h2("4.1 [Use Case Name]"),
      body("[Present-tense, agent-centric description. No editorializing — caveats go in Open Items.]"),
      spacer(),
      h2("4.2 [Use Case Name]"),
      body("[Present-tense, agent-centric description.]"),
      spacer(),

      // 5. Integration Architecture
      h1("5. Integration Architecture"),
      body("[System 1]: [what it does, what data flows through it]."),
      body("[System 2]: [what it does, what data flows through it]."),
      spacer(),

      // 6. Phases & Deliverables
      h1("6. Phases & Deliverables"),
      phasesTable([
        { phase: "Discovery", description: "Requirements gathering, integration mapping", deliverables: "Requirements doc, integration spec" },
        { phase: "Build", description: "Agent development, integration setup", deliverables: "Working agent in sandbox" },
        { phase: "UAT", description: "Client testing, refinement", deliverables: "Test results, sign-off" },
        { phase: "Go-Live", description: "Production deployment, monitoring", deliverables: "Live agent, monitoring dashboard" },
        { phase: "Hypercare", description: "Post-launch support, tuning", deliverables: "Performance reports" }
      ]),
      spacer(),

      // 7. Roles & Responsibilities
      h1("7. Roles & Responsibilities"),
      rolesTable([
        { responsibility: "Agent design & build", polyai: "Owner", client: "Reviewer" },
        { responsibility: "Integration credentials & API access", polyai: "—", client: "Provider" },
        { responsibility: "Knowledge base content", polyai: "Drafter", client: "Approver" },
        { responsibility: "UAT testing", polyai: "Support", client: "Owner" },
        { responsibility: "Production telephony setup", polyai: "Guide", client: "Owner" }
      ]),
      spacer(),

      // 8. Open Items
      h1("8. Open Items"),
      openItemsTable([
        { item: "[Short label]", description: "[Specific open question]", owner: "[Client/Partner/PolyAI]" }
      ]),
      spacer(),

      // 9. Future State
      h1("9. Future State"),
      body("[Next-phase vision, clearly labeled as future/not-committed scope.]"),
    ]
  }]
});

Packer.toBuffer(doc).then(buf => {
  fs.writeFileSync(outputPath, buf);
  console.log(`Saved to ${outputPath}`);
});
```

## Notes

- Font is Arial throughout; body text at size 21 (10.5pt), bullets at 20 (10pt), table cells at 19 (9.5pt).
- Headings use `h1` for numbered top-level sections (1-9) and `h2` for use case subsections (4.1, 4.2, ...).
- Pass the output path as a CLI argument: `node generate.js ~/Desktop/ClientName_Scoping.docx`
- For expansion docs, change section 2 heading to "Current Deployment" and add use case tags (New/Enhancement/Migration).
