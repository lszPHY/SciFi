---
Rank: 2
Timeout: 1800
BashTime: 600
Skills: local_env
ForceModel: deepseekv4pro
PrescanModel: claude-opus4.7
ReviewModel: gpt5.5
CommonStorage: rw
TaskGroup: miniDAQ
---

# MiniDAQ: Decode run00131 and Plot ADC/TDC Spectra

## Context
Process a MiniDAQ raw data file and produce standard diagnostic spectra (TDC overall, TDC per channel, ADC overall, ADC per channel, channel hits), following the conventions of the reference project at https://github.com/lszPHY/phase2_MiniDAQ.git.

The reference project is the authoritative source for raw word format, decode algorithm, tube/channel geometry and mapping, ADC/TDC field extraction, and plotting conventions. The agent must NOT invent its own decode logic and must NOT use ROOT — use only what the reference code uses (numpy, struct, matplotlib).

Raw data file: /mnt/run00131_20260309_164054.dat

Decomposition: repo.summarize.md (clone + notes), data.decode.md (decode .dat to .npz), plot.spectra.md (5 PNGs). All outputs land in the task output directory.

## Todo
1. Verify the raw data file exists with ls -lh /mnt/run00131_20260309_164054.dat. If missing, write error.txt and exit non-zero.
2. Set up a local Python env per the local_env skill: create ./mamba_env with python=3.12, numpy, matplotlib, and write env.sh. Subtasks may install extras on demand.
3. Run subtask repo.summarize.md to produce miniDAQ_notes.md.
4. Run subtask data.decode.md to produce decoded.npz.
5. Run subtask plot.spectra.md to produce the 5 PNG figures.
6. Write SUMMARY.md listing each output file with its size and a one-line description, plus the total number of decoded hits.

## Expect
- miniDAQ_notes.md exists, non-empty, contains sections on tube geometry, decode process, and plotting conventions.
- decoded.npz exists and loads cleanly with numpy; contains per-hit arrays for channel, ADC, and TDC; total hit count > 0.
- All 5 PNG files exist in the task output directory and are each > 10 KB: tdc_overall.png, tdc_per_channel.png, adc_overall.png, adc_per_channel.png, channel_hits.png.
- SUMMARY.md exists and lists every output file above.
- No subtask failed.
