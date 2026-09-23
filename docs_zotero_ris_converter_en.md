# 🛠️ Developer Documentation: Zotero RIS ➔ Obsidian Two-Pole Card Converter

> **Language Switch:** [Magyar nyelvű dokumentáció](docs_zotero_ris_converter.md) | **Standalone Web App:** [Zotero to Obsidian Converter (EN)](public/zotero_ris_to_obsidian_en.html) / [Magyar felület](public/zotero_ris_to_obsidian.html)

**Source File:** `public/zotero_ris_to_obsidian_en.html`  
**Architecture:** Zero-dependency, Client-Side Vanilla HTML5 + CSS3 + ES6 JavaScript  
**Core Purpose:** Automated, lossless knowledge transfer from Zotero to an Obsidian Zettelkasten vault using a bi-directional two-pole card model (`bib_*` ⟷ `src_*_notes`).

---

## 1. System Philosophy: The Two-Pole Card Model

In Personal Knowledge Management (PKM) and academic research workflows, researchers typically encounter two pitfalls:
1. **The Monolith Trap:** Overcrowding a single bibliographic note with all reading highlights, quotes, and fleeting thoughts, rendering it cluttered and unreadable.
2. **The Disconnection Trap:** Completely separating notes from source records, causing citations and deep links back to the original text to be lost over time.

This tool resolves both issues by establishing a synchronized **Two-Pole Card Pair** for every reference item:

```
┌──────────────────────────────────────┐          ┌──────────────────────────────────────┐
│  Pole 1: Bibliographic Anchor        │          │  Pole 2: Research & Reading Notes    │
│  File: bib_jour_kovacs2026trans.md   │ ◄──────► │  File: src_kovacs2026trans_notes.md  │
├──────────────────────────────────────┤          ├──────────────────────────────────────┤
│ • Pure, standard citation metadata   │          │ • PDF annotations & highlights       │
│ • Google Drive linked PDF path       │          │ • Clickable deep links (p. 42)       │
│ • Pandoc key: [@kovacs2026trans]     │          │ • Personal reflection & synthesis    │
│ • Linked notes: [[src_..._notes]]    │          │ • Dataview query for concept atoms   │
└──────────────────────────────────────┘          └──────────────────────────────────────┘
                                      ▲
                                      │ (Final Publication "Goal")
                                      ▼
             Word / Pandoc / Typst with Zotero Integration
```

1. **Bibliographic Anchor (`bib_*`):**
   - Serves as the authoritative source identity card.
   - Strictly validated YAML frontmatter (14-digit timestamp ID, author, year, title, publisher, journal, volume, issue, pages, DOI, local Google Drive link).
   - Zotero Better BibTeX Pandoc citekey (`[@citekey]`) ready for final thesis insertion and automated bibliography compilation.
   - Direct link to its partner note: `notes_card: "[[src_*_notes]]"`.

2. **Reading Notes & Annotations (`src_*_notes`):**
   - Serves as the researcher's cognitive workspace during reading.
   - Frontmatter link pointing back to the source: `source_bib: "[[bib_*]]"`.
   - Contains extracted Zotero quotes, deep links (`zotero://open-pdf/...`), and student reflections.
   - Embedded Dataview block tracking all atomic concept cards (`concept_*`) derived from this study.

---

## 2. RIS Tag Mapping Matrix

The engine parses standard RIS tags line by line:

| RIS Tag | Semantic Field | Extracted Value | Target Obsidian Field |
| :--- | :--- | :--- | :--- |
| `TY  - ...` | Publication Type | Evaluated against `RIS_TYPE_MAP` | `doc_type` (`bib_jour`, `bib_book`, etc.) |
| `AU` / `A1` | Authors | Array joined with semicolons | `author` |
| `PY` / `Y1` | Publication Year | 4-digit regex extraction | `year` |
| `TI` / `T1` | Title | Sanitized title string | `title` |
| `JO` / `JF` | Journal Name | Extracted for journal articles (`bib_jour`) | `publication` |
| `BT` / `T2` | Book / Proceedings Title | For chapters (`bib_chap`) or conferences (`bib_conf`) | `booktitle` |
| `PB` | Publisher / Institution | Publishing press or academic institution | `publisher` |
| `VL` | Volume | Journal volume number | `volume` |
| `IS` | Issue | Journal issue number | `issue` |
| `SP` / `EP` | Pages | Concatenated page span (`SP-EP`) | `pages` |
| `DO` | DOI | Cleaned digital object identifier | `doi` and hyperlinked DOI URL |
| `UR` | Web URL | Official web link | `url` |
| `AB` / `N2` | Abstract | Formatted multi-line blockquote (`> `) | `## 📑 Official Abstract` |
| `N1` / `RN` | Notes / Annotations | Extracted Zotero highlights | `src_*_notes` markdown body |
| `L1` | Local File Path | Linked Google Drive PDF location | `archive` |

---

## 3. Core Algorithms

### A) Collision-Free 14-Digit Timestamp ID (`YYYYMMDDHHmmss`)
The PKM Lego standard requires a human-readable, chronological 14-digit identifier. To avoid ID collisions during batch exports:
- The `bib_*` card receives `offset = index * 2` seconds.
- The corresponding `src_*_notes` card receives `offset = index * 2 + 1` seconds.

### B) Better BibTeX-Compliant Citekey Generation
The `cleanAscii(str)` function normalizes UTF-8 text using Unicode NFD decomposition, strips all diacritics, and removes non-alphanumeric characters:
```
Formula: [First Author's Clean Surname][4-Digit Year][First Substantive Title Word]
Example: Kovács, János (2026): A transzformatív tanulás... ➔ kovacs2026transzformativ
```

---

## 4. Academic Methodology: The 4-Color Highlighting System

The application embeds the internationally recognized 4-color highlighting protocol for scientific PDF appraisal in Zotero:
- 🟡 **Yellow (Core Concepts & Definitions):** Seed material for creating atomic `concept_*` cards.
- 🔵 **Blue (Methodology & Empirical Data):** Sampling, research design, statistical indicators, test setups.
- 🟢 **Green (Findings & Key Takeaways):** Authors' verified claims, answers to research questions.
- 🔴 **Red (Critique & Open Questions):** Methodological limitations, counter-arguments, researcher doubts.

---

## 5. Deployment & Zero-Install Distribution

1. **Standalone Offline File:** `public/zotero_ris_to_obsidian_en.html` runs in any desktop or mobile web browser with zero network access required.
2. **GitHub Pages Ready:** Serve directly from a repository's `public/` or `docs/` folder.
3. **Privacy First:** 100% of data processing occurs in local browser memory; no data or files are transmitted to external servers.
