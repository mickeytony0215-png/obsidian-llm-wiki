# raw/

The "raw sources" layer of the wiki — immutable original documents (paper PDFs, extracted text, web clippings). **Contents are gitignored** because they are third-party copyrighted material.

Expected subfolders:

- `raw/papers/` — paper PDFs and extracted `*_text.txt` files used as LLM context
- `raw/web/` — web-clipped articles and notes

The wiki under `../wiki/sources/` references these files by filename, so keep filenames stable once a source page links to them.
