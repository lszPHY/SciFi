---
Rank: 1
Timeout: 1800
BashTime: 600
Skills: local_env
---

# Plot ADC/TDC Spectra and Channel Hits

## Context
Produce the standard MiniDAQ diagnostic plots from decoded.npz, using the binning, axis ranges, per-channel subplot layout, and scaling conventions documented in miniDAQ_notes.md and as implemented in /tmp/phase2_MiniDAQ.

Output exactly 5 PNG files in the task output directory: tdc_overall.png (1-D histogram of all TDC values), tdc_per_channel.png (grid of per-channel TDC histograms), adc_overall.png (1-D histogram of all ADC values), adc_per_channel.png (grid of per-channel ADC histograms), channel_hits.png (bar/line plot of hit count vs channel).

Use matplotlib with the Agg backend so no display is needed. Do NOT use ROOT.

## Todo
1. Verify decoded.npz and miniDAQ_notes.md exist; exit non-zero if not.
2. Activate the env with source env.sh. Install matplotlib if not present.
3. Write plot_spectra.py that loads decoded.npz, reads binning and range conventions from miniDAQ_notes.md (hard-code them in the script with a comment citing the section), cross-checks the per-channel grid shape against the reference code in /tmp/phase2_MiniDAQ, and produces the 5 PNGs listed above. Each figure should have a title, axis labels, and per-channel subplots should be annotated with their channel number.
4. Run python plot_spectra.py 2>&1 | tee plot.log.
5. Verify each PNG exists and is non-trivial in size by listing them with ls -l.

## Expect
- plot_spectra.py exists.
- All 5 PNG files exist in the task output directory: tdc_overall.png, tdc_per_channel.png, adc_overall.png, adc_per_channel.png, channel_hits.png.
- Each PNG is larger than 10 KB.
- plot.log exists and contains no Python traceback.
- The per-channel figures contain one subplot per channel that has at least one hit, matching the channel count documented in miniDAQ_notes.md.
