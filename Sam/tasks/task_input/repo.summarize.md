---
Rank: 2
Timeout: 6000
BashTime: -1
Skills: local_env
---

# Summarize phase2_MiniDAQ Reference Project

## Context
Clone the reference repository and extract everything a continuing agent needs to decode raw MiniDAQ data and plot ADC/TDC spectra in the project's style.

Repo: https://github.com/lszPHY/phase2_MiniDAQ.git

Clone to /tmp/phase2_MiniDAQ (reference only, not a persistent artifact). The summary file miniDAQ_notes.md MUST be written to the task output directory (current working directory) and will be consumed by the decode and plotting subtasks.

## Todo
1. Run git clone https://github.com/lszPHY/phase2_MiniDAQ.git /tmp/phase2_MiniDAQ. If the clone fails, retry once; if it still fails, write the error to clone_error.txt and exit non-zero.
2. List the repo structure with find /tmp/phase2_MiniDAQ -maxdepth 3 -type f and filter for code/doc files (py, md, ipynb, txt, cpp, h).
3. Read the README and any top-level docs.
4. Locate and read decode-related scripts (anything mentioning decode, raw, unpack, parse, dat, word size, byteorder, header/trailer). Identify word size, endianness, header/trailer markers, and how channel, ADC, and TDC are extracted (bit masks/shifts).
5. Locate and read plotting scripts (ADC, TDC, channel hits, occupancy). Note histogram binning, axis ranges, per-channel subplot grid shape, log/linear scaling.
6. Locate geometry/channel-mapping definitions (tube layout, channel-to-tube mapping, total number of channels).
7. Write miniDAQ_notes.md in the task output directory with these section headings exactly: # MiniDAQ Reference Notes, ## Repository Layout, ## Tube Geometry and Channel Mapping, ## Raw Data Format, ## Decode Process, ## ADC and TDC Extraction, ## Plotting Conventions, ## Caveats and Gotchas, ## Source Files Referenced. For each section, cite specific file(s) and line ranges in the repo. If information is genuinely missing from the repo, state so explicitly rather than guessing.

## Expect
- miniDAQ_notes.md exists in the task output directory.
- File size > 2 KB.
- Contains all 8 required section headings exactly as listed.
- The ## Decode Process section explicitly states the word size in bytes and the bit layout (or masks/shifts) for channel, ADC, and TDC.
- The ## Plotting Conventions section states histogram bin counts and axis ranges used in the reference project for both ADC and TDC.
- The ## Source Files Referenced section lists at least 2 files from the cloned repo with paths.
