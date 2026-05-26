---
Rank: 2
Timeout: 1800
BashTime: -1
Skills: local_env
---

# Decode run00131 Raw Data

## Context
Decode /mnt/run00131_20260309_164054.dat into structured per-hit arrays, following exactly the algorithm documented in miniDAQ_notes.md (produced by repo.summarize.md) and as implemented in the reference repo cloned at /tmp/phase2_MiniDAQ.

Do NOT invent decode logic. If miniDAQ_notes.md is missing or the decode section is incomplete, re-consult /tmp/phase2_MiniDAQ directly. Do NOT use ROOT.

Output decoded.npz with at minimum three equal-length 1-D numpy integer arrays: channel, adc, tdc. Additional arrays (event id, timestamp, etc.) may be included if the reference decode produces them.

## Todo
1. Verify input file with ls -lh /mnt/run00131_20260309_164054.dat. If missing, write error.txt and exit non-zero.
2. Verify miniDAQ_notes.md exists in the task output directory; exit non-zero if not.
3. Activate the env with source env.sh. If numpy is not installed, install it via the env's pip.
4. Write decode.py that opens the raw file in binary mode, applies the word format from miniDAQ_notes.md (cross-check /tmp/phase2_MiniDAQ source if anything is unclear), extracts channel/adc/tdc per hit using the exact masks/shifts from the reference, handles header/trailer words as the reference does, saves arrays via numpy.savez_compressed to decoded.npz, and prints total hits with min/max for channel, ADC, TDC.
5. Run python decode.py 2>&1 | tee decode.log.
6. Sanity check: load decoded.npz and print the shape, dtype, and min/max for each array.

## Expect
- decode.py exists.
- decoded.npz exists and loads with numpy without error.
- decoded.npz contains arrays named channel, adc, tdc.
- The three arrays have identical length, and that length is > 0.
- channel.min() >= 0 and channel.max() is consistent with the channel count documented in miniDAQ_notes.md.
- decode.log exists and contains the printed totals/min/max line.
