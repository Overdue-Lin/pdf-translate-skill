---
name: pdf-translator
description: Translate PDF documents while preserving original formatting. Converts PDF pages to images for layout analysis, extracts text and embedded images, translates content using agent's multilingual capabilities, and generates LaTeX code to recreate the original document structure with translated text.
---

# PDF Translator Skill

A comprehensive skill for translating PDF documents while maintaining their original layout, formatting, and visual structure. This skill combines multimodal analysis, text extraction, machine translation, and LaTeX typesetting to produce high-quality translated PDFs.

## When to Use This Skill

Use this skill when you need to:
- Translate PDF documents from one language to another
- Preserve the original document's visual layout and formatting
- Handle documents with mixed text and images
- Process academic papers, reports, or formatted documents
- Support Chinese/English bidirectional translation

## Prerequisites

### Required Software
- **Python 3.8+** with pip
- **PyMuPDF** (`pip install pymupdf pillow`)
- **XeLaTeX** (TeX Live or MiKTeX) for PDF compilation
- **ctex LaTeX package** for Chinese support

### Installation Commands
```bash
# Python dependencies
pip install pymupdf pillow

# LaTeX (choose one)
# Ubuntu/Debian:
sudo apt install texlive-xetex texlive-lang-chinese

# macOS (Homebrew):
brew install mactex

# Windows: Download MiKTeX from https://miktex.org/
```

## Workflow Overview

The PDF translation follows this pipeline:

```
PDF Input → Page Images → Layout Analysis → Text Extraction → Translation → LaTeX Generation → PDF Output
```

### Step 1: Convert PDF to Page Images

Convert each PDF page to a high-quality image for multimodal analysis.

```bash
python scripts/pdf_to_images.py input.pdf output_pages/ --dpi 150
```

**Output:**
- `page_001.png`, `page_002.png`, ... (page images)
- `manifest.json` (page dimensions and metadata)

### Step 2: Extract Embedded Images

Extract original images from the PDF for reuse in the translated document.

```bash
python scripts/extract_images.py input.pdf output_extracted/
```

**Output:**
- `images/img_001_01.png`, ... (extracted images)
- `images_manifest.json` (image positions and metadata)

### Step 3: Analyze Layout and Extract Content

For each page image, analyze the layout and extract content:

**Using multimodal analysis:**
1. Describe the page layout structure (columns, sections, headers, footers)
2. Identify text blocks, their positions, and reading order
3. Identify figures, tables, and their captions
4. Note any special formatting (lists, equations, quotes)

**Prompt template for layout analysis:**
```
Analyze this PDF page image and describe:
1. Overall layout structure (columns, margins, alignment)
2. Text blocks with their approximate positions and hierarchy
3. Images, figures, or tables with their locations
4. Any special elements (headers, footers, page numbers, footnotes)

Provide a structured description that can be used to recreate this layout in LaTeX.
```

### Step 4: Translate Text Content

Translate the extracted text while maintaining:
- Technical terms accuracy
- Proper nouns (names, places)
- Formatting markers (emphasis, bold)
- Contextual meaning

**Translation guidelines:**
- Preserve original paragraph structure
- Maintain consistent terminology
- Handle inline formatting (bold, italic, code)
- Keep mathematical formulas unchanged

### Step 5: Generate LaTeX Code

Create LaTeX code that recreates the original layout with translated content.

**Key considerations:**
1. **Document class**: `article` for papers, `report` for longer documents
2. **Page geometry**: Match original margins
3. **Multi-column layouts**: Use `multicol` package
4. **Figures**: Use `\includegraphics` with proper positioning (Note: Insert the extracted images into their original positions)
5. **Tables**: Recreate with `tabular` or `longtable` (Note: Insert the identical table at its original position)
6. **Chinese support**: Always use `ctex` package with XeLaTeX
7. **!IMPORTANT**: Please modify the template format to match the style of the original PDF (Note: Pay special attention to the switching between single-column and double-column layouts)

**!IMPORTANT**: You can refer to **./references/latex_template.md** for details related to LaTeX generation.

**Template structure:**
```latex
\documentclass[12pt, a4paper]{article}
\usepackage{ctex}  % Chinese support
\usepackage{graphicx}
\usepackage{geometry}
% ... other packages

\begin{document}
% Translated content here
\end{document}
```

### Step 6: Compile LaTeX to PDF

Compile the generated LaTeX code to produce the final translated PDF.

```bash
python scripts/compile_latex.py translated.tex output_dir/ --passes 2
```

**Output:**
- `translated.pdf` (final translated document)

If any errors occur, you can troubleshoot them by referring to **./references/troubleshooting.md**.

## Detailed Implementation Guide

### Phase 1: Preparation

1. **Create working directory structure:**
   ```
   working_dir/
   ├── input.pdf
   ├── pages/           # Page images
   ├── extracted/       # Extracted images
   ├── translated.tex   # Generated LaTeX
   └── output.pdf       # Final result
   ```

2. **Convert PDF to images:**
   ```
   Use: python scripts/pdf_to_images.py
   Output: pages/page_001.png, ..., pages/manifest.json
   ```

3. **Extract embedded images:**
   ```
   Use: python scripts/extract_images.py
   Output: extracted/images/*, extracted/images_manifest.json
   ```

### Phase 2: Page-by-Page Processing

For each page:

1. **View page image** using multimodal capability

2. **Analyze layout structure:**
   - Number of columns
   - Section headings
   - Paragraph blocks
   - Figure/table positions
   - Header/footer content

3. **Extract text content:**
   - Use vision to identify text regions
   - Transcribe text from image (OCR-like)
   - Note formatting (bold, italic, font sizes)

4. **Translate content:**
   - Translate each text block
   - Maintain structure and formatting
   - Handle technical terms appropriately

5. **Generate LaTeX for page:**
   - Apply appropriate layout template
   - Insert translated text
   - Include images with correct positions
   - Handle special elements (tables, equations)

### Phase 3: Document Assembly

1. **Combine all pages into single LaTeX file:**
   - Use base_template.tex as foundation
   - Insert page content in order
   - Handle cross-page elements (tables, figures)

2. **Compile to PDF:**
   ```
   Use: python scripts/compile_latex.py
   Run multiple passes for references
   ```

3. **Verify output:**
   - Check all pages rendered correctly
   - Verify images are included
   - Confirm formatting matches original

## Common Layout Patterns

### Single Column Document
```latex
\section{Section Title}
Translated paragraph content here.

\begin{figure}[htbp]
  \centering
  \includegraphics[width=0.8\textwidth]{image.png}
  \caption{Translated caption}
\end{figure}
```

### Two Column Layout
```latex
\begin{multicols}{2}
\section{Left Column}
Content...

\section{Content Spanning}
...
\end{multicols}
```

### Academic Paper Structure
```latex
\title{Translated Title}
\author{Authors}
\maketitle

\begin{abstract}
Translated abstract...
\end{abstract}

\section{Introduction}
Content...

\section{Methods}
Content...

\bibliographystyle{plain}
\bibliography{references}
```

## Reference Files

- `assets/base_template.tex` - Base LaTeX template with Chinese support
- `references/latex_templates.md` - LaTeX code patterns for various layouts
- `references/troubleshooting.md` - Common issues and solutions

## Troubleshooting

### Font Issues
If Chinese characters don't render:
- Ensure XeLaTeX is used (not pdfLaTeX)
- Check ctex package is installed
- Verify system has Chinese fonts installed

### Image Not Found
If images don't appear in output:
- Check `\graphicspath` includes correct directory
- Verify image files exist and are readable
- Use correct file extensions in `\includegraphics`

### Layout Mismatch
If layout differs from original:
- Adjust geometry settings for margins
- Use appropriate figure placement options
- Check for column count differences

### Compilation Errors
If LaTeX compilation fails:
- Check log file for specific errors
- Verify all required packages are installed
- Ensure no syntax errors in generated code

## Best Practices

1. **Quality over speed**: Take time to analyze layout accurately
2. **Iterative refinement**: Generate, compile, review, and improve
3. **Preserve semantics**: Maintain document meaning over exact formatting
4. **Handle edge cases**: Be prepared for tables, equations, code blocks
5. **Test compilation**: Verify LaTeX compiles before final output

## Limitations

- **Exact pixel-perfect replication**: LaTeX is not a pixel-based layout engine
- **Complex graphics**: Vector illustrations may need to be captured as images
- **Custom fonts**: May require font substitution
- **Dynamic content**: JavaScript or form fields are not supported
- **Encrypted PDFs**: Cannot process password-protected documents

## Output Verification Checklist

- [ ] All pages included in output
- [ ] Text correctly translated
- [ ] Images positioned correctly
- [ ] Tables formatted properly
- [ ] No LaTeX compilation errors
- [ ] Chinese characters render correctly
- [ ] Page numbers and headers present
- [ ] Overall formatting matches original
