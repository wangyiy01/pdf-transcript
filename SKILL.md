---
name: pdf-transcript
description: "逐字稿：把截图型、扫描型或从网页/飞书导出的 PDF 转成易读版逐字稿文档。Use when the user asks to 提取逐字稿、转录 PDF、截图 PDF 转文字、写入飞书文档、整理成易读文档, especially when pages contain screenshots, PPT images, tables, diagrams, colored/bold text, or mixed正文+图片区域. If the user does not explicitly request raw/original-only output, default to a readable document, not a plain OCR dump."
---

# 逐字稿

## Default

When the user provides a screenshot-like PDF and asks for a transcript, default to an **易读版逐字稿文档** unless they explicitly say `原模原样`, `不要排版`, `纯文本`, or `只要 OCR 原文`.

Readable means:

- Preserve the original content and order; do not summarize, paraphrase, or add new arguments.
- Improve structure with headings, paragraph breaks, bold emphasis, colors, and images.
- Treat diagrams/tables in images as visual material: crop and insert the image, or redraw as a table/diagram when that is more readable.

## Workflow

1. **Inspect the PDF**
   - Check whether it has a text layer.
   - If it is image-only or screenshot-heavy, render every page to high-resolution page images.
   - Keep page numbers throughout the pipeline.
   - Check whether one PDF contains multiple independent documents/articles before writing:
     - Use title pages, big section markers, chapter resets, outline structure, page-level headings, and naming patterns such as `A.`, `B.`, `C.`, `第一篇`, `第二篇`, or a new top-level title to detect boundaries.
     - When distinct documents can be inferred from titles and hierarchy, split them into separate target documents/files by default instead of merging them into one transcript.
     - Remove cross-boundary residue: text before the selected document's title belongs to the previous document; text after the next document's title belongs to the next document.
     - Preserve each split document's own heading hierarchy and figures; do not duplicate a shared/manual目录 across the split outputs.

2. **OCR with layout**
   - Extract both plain text and line-level positions if possible.
   - Keep OCR line boxes so正文区域, picture/table regions, headers, footers, and page artifacts can be separated.
   - Do not treat “this page is an image” as a reason to skip the whole page. Skip only image/table/diagram regions when instructed to skip images.

3. **Classify content**
   - Keep formal article content: title, speaker/author information, and正文 from the real opening.
   - For multi-document PDFs, classify content within each detected document boundary independently. Do not let glossary tails, FAQ tails, figure OCR, or the next article's opening leak into the current output.
   - Drop non-article artifacts by default:
     - `C2`, `C-2`, source-system labels, creator/modifier/time metadata.
     - Manually copied directory/table-of-contents blocks when the target document can generate an outline from headings.
     - Footer clutter such as `仅供内部`, `未经授权`, `往期大咖`, recommended-reading lists, learning-map links.
   - If uncertain whether a block is article正文 or metadata, keep it and flag it briefly.

4. **Handle visual material**
   - For tables, charts, architecture diagrams, PPT screenshots, and structured figures:
     - Prefer cropping the original visual and inserting it at the relevant position.
     - Redraw as a table/diagram only when it is clearer than the screenshot and does not change meaning.
   - Do not paste OCR fragments from figures as prose. This creates unreadable scattered text.
   - Keep useful figure titles/captions unless the user asks to remove them.

5. **Build the readable transcript**
   - Reconstruct heading hierarchy from the source.
   - Preserve visible styling signals:
     - Colored headings or emphasized source text should remain colored when the target supports it.
     - Bold source text or obvious lead-in sentences should remain bold.
     - Maintain section hierarchy instead of flattening everything into paragraphs.
   - Merge OCR line wraps into natural sentences.
   - Split long paragraphs by topic and sentence boundaries. Aim for roughly 250-400 Chinese characters per paragraph; avoid paragraphs above about 420 characters.
   - Do not convert narrative sections into bullets unless the source is already a list or a row/column structure.

6. **Write to the requested target**
   - If the user has not specified a target location, ask them which output target to use before writing.
   - For Feishu/Lark output, specifically remind the user to choose the Feishu Drive folder where the new transcript document(s) should be saved. If the PDF is split into multiple documents, use the same chosen folder for all split outputs unless the user says otherwise.
   - For Feishu/Lark Docx:
     - Use structured XML when possible.
     - Use headings for the article outline; do not duplicate a manual目录.
     - Insert cropped/redrawn visuals near their source location.
     - If overwriting the document, remember that image blocks may be removed; reinsert visuals afterward.
   - For local deliverables, save polished outputs under the user-facing outputs directory when appropriate.

7. **Validate before final response**
   - Check document splitting: if the source PDF appears to contain multiple articles/documents, confirm each output starts at its own formal title/opening and ends before the next formal title/opening.
   - Check section outline: no missing major sections, no accidental manual目录.
   - Check split boundaries: later document headings should not appear in earlier outputs, and earlier document headings should not appear in later outputs.
   - Check source cleanup: no `C2`, creator/modifier metadata, internal footer, or往期推荐 remains unless requested.
   - Check images: expected cropped/redrawn visuals are present and in the right order.
   - Check readability: no remaining very long paragraphs; paragraphs should not be one OCR line each.
   - Check formatting: colors, bold, and heading levels are preserved where useful.
   - Mention any OCR uncertainty or low-confidence visual reconstruction.

## Modes

- **Default readable mode**: faithful content plus improved document structure.
- **Raw transcript mode**: use only when explicitly requested. Output OCR text with minimal cleanup; skip added headings, colors, figure crops, and paragraph restructuring beyond necessary line joins.
- **Image-skip mode**: when the user says images should be skipped, skip only picture/table/diagram regions, not whole PDF pages.

## Feishu/Lark Notes

- Use the `lark-doc` skill for Feishu doc creation/editing.
- Use document headings instead of a hand-copied目录.
- Keep figure captions only when they help reading; if they pollute the outline, consider making them normal paragraphs instead of heading blocks.
- After every write, fetch a small outline and keyword samples to verify formatting and image placement.
