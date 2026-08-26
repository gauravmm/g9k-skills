---
name: md2paper
description: Render markdown to a PDF sized for a Viwoods AiPaper e-ink tablet, or send an existing PDF (optionally cropped to 4:3), and upload it to the tablet's Learning section over LAN, for reading specs and long documents away from the desk.
---

# Skill: Send Markdown to a Viwoods AiPaper

Use this when the task is to put a written document on the AiPaper tablet to read: a spec, a design doc, a long report.

`md2paper` is bundled beside this file. Copy it to a directory on `PATH` and make it executable.

```sh
install -m 755 md2paper ~/.local/bin/md2paper
```

It runs under `uv` through a PEP 723 header, so it resolves its own dependencies on first run. Nothing needs installing into a project.

## Usage

```sh
md2paper spec/DESIGN.md          # render, then upload to Learning
md2paper notes.pdf               # send a PDF as it stands
md2paper --trim notes.pdf        # crop to 4:3, stand landscape pages up, then send
md2paper -n -o out/ *.md         # render only, keep the PDFs
md2paper --list                  # Learning folders on the device
md2paper --folder Specs a.md     # pick the destination folder by name
```

`--folder` is only needed when the tablet has more than one folder under Learning. `MD2PAPER_FOLDER` sets a default. `VIWOODS_HOST` overrides the device address, which defaults to `http://viwoods-aipaper:8090`.

## What It Does

- Renders through `markdown-it-py` and `weasyprint` onto a 159x212 mm page, the screen's 3:4 shape at its physical size, so the tablet shows the type at the size it was set in.
- Sets a serif body against sans headings, justified with hyphenation, and code that wraps instead of running off the page.
- Uploads over the device's chunked `/upload_chunk` protocol into the Learning section, which is the only section that accepts a PDF.
- A PDF is uploaded as-is. `--trim` (`--crop`) stands landscape pages up and crops each page to 4:3 (tall:wide) first — a box change, not a raster — and does not touch the source file.

## Facts Worth Knowing First

- **Every upload is permanent.** The device has no delete verb and no overwrite. A second push of the same file creates `Name(1)` beside it, and only a hand on the tablet removes either.
- **The LAN Transfer app must be in the foreground.** A woken tablet alone refuses the connection.
- **A sleeping tablet does not block.** The push renders, detaches a retry loop, prints its pid and log path, and retries every minute until the file lands. Kill it by pid to stop it.
- The script refuses to run when the fonts it sets type in are absent, and prints the `apt` line that supplies them.

## When Not To Use It

Do not use this to move a file the user has not asked to read on paper. Each upload is permanent and costs device storage that only a human can reclaim.
