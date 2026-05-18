---
Rank: 2
BashTime: -1
Skills: common_env
GPU: no
CommonStorage: rw
TaskGroup: miniDAQ
PrescanModel: claude-opus-4-7
ForceModel: deepseek-v4-pro
ReviewModel: gpt-5.5
---

# MiniDAQ delta_t debug and trigger-match verification

## Context
This is an ongoing physics DAQ analysis project located at
`~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ`. **Preserve all existing
work** — do not delete, rewrite, or reorganize files. Read what is there
first and continue from the current state.

**Raw data file** (input, do not modify):
`~/run00131_20260309_164054.dat`

**Reference repositories** (clone fresh into `./reference/` inside the
project directory):
- `https://github.com/lszPHY/phase2_MiniDAQ.git` — the `decode` logic
  here is the ground-truth reference. Compare the current project's
  decoder against it and fix discrepancies.

if any address is appropriate, stop immediately and report whats missing.

**Key concepts**:
- **Fitted delta_t**: drift time extracted from a C++ linear fit of the
  digitized trace (existing `cppfit` code in the project). The
  difference between consecutive fitted trigger times gives a fitted
  delta_t per event pair.
- **PRC delta_t**: delta_t computed from PRC hardware timestamps
  recorded per event.
- Both distributions should peak in **~200–300 ns** (acceptable
  window: 150–350 ns). A peak outside this window indicates a decoding
  or unit-conversion bug.
- **Trigger-match event**: an event where
  `|fitted_trigger_time − PRC_trigger_time| < 50 ns`.

**Environment**: use `common_env` skill. Likely needs Python
(numpy, scipy, matplotlib) and possibly ROOT / a C++ compiler for the
existing `cppfit`. First check `/mnt/sci_envs/` for an existing env
(look for names like `root`, `minidaq`, `physics`); only create a new
env if none is suitable. **Never modify an existing shared env.**


**Deliverables** (all written to the project root
`~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ/`):
- `report.md`
- `delta_t_hist.png`
- `trigger_time_agreement.png`

## Todo
1. `cd ~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ` and list the
   directory. Read `README*`, top-level scripts, and any existing
   analysis entry points to understand the current state. Identify the
   decoder, the `cppfit` linear-fit code, the PRC-time extractor, and
   any existing delta_t computation.
2. Discover or create a suitable env via the `common_env` skill
   (numpy, scipy, matplotlib; add compilers if the existing
   pipeline requires them). Record the env path used.
3. Create `./reference/` and clone both reference repos there:
   - `git clone https://github.com/lszPHY/phase2_MiniDAQ.git reference/phase2_MiniDAQ`
   - `git clone https://github.com/lszPHY/SciFi.git reference/SciFi`
   Read `reference/SciFi/README.md` and the decoder source under
   `reference/phase2_MiniDAQ/`.
4. Compare the current project's decoder against
   `reference/phase2_MiniDAQ/`. Note every discrepancy that could
   affect trigger time, PRC time, or delta_t (byte offsets, word
   order, clock unit conversion, sign, masking, etc.). Document each
   finding with file:line references.
5. For each real bug identified, apply a minimal fix in the current
   project's code. Keep changes surgical; do not refactor unrelated
   code. Record the diff (file path + before/after snippet) for the
   report.
6. Build (if needed) and run the project's pipeline on
   `~/run00131_20260309_164054.dat` end-to-end. The pipeline must
   produce, per event: (a) the cppfit-derived trigger time, and
   (b) the PRC trigger time. Use BashTime: -1 — the run may be long.
7. From the per-event outputs, compute:
   - `fitted_delta_t[i] = fitted_trigger_time[i+1] − fitted_trigger_time[i]`
   - `prc_delta_t[i]    = prc_trigger_time[i+1]    − prc_trigger_time[i]`
   Histogram both (overlaid) in nanoseconds. Fit/locate the peak of
   the fitted_delta_t distribution. Save plot to `delta_t_hist.png`
   (x-axis in ns, range covering at least 0–600 ns, with the peak
   value annotated).
8. Compute per-event difference
   `diff[i] = fitted_trigger_time[i] − prc_trigger_time[i]` (after
   aligning to a common zero / removing any constant offset if the
   existing code defines one — document the alignment choice).
   Plot a scatter of fitted vs PRC trigger time and a histogram of
   `diff` in `trigger_time_agreement.png`. Report mean and std of
   `diff`.
9. Count `N_match = number of events with |diff| < 50 ns`. Also
   report the total number of events processed and the matched
   fraction.
10. Write `report.md` in the project root containing the following
    sections, with concrete numbers (not placeholders):
    - **Bugs found and fixes** — list each bug with file:line and the
      applied fix; if none, write "No bugs found" and justify by
      showing the decoder matches the reference.
    - **Delta_t result** — fitted_delta_t peak value in ns and whether
      it falls in 150–350 ns; PRC_delta_t peak for comparison.
    - **Trigger match** — total events, `N_match`, matched fraction,
      mean and std of `diff` in ns, and a verdict on whether fitted
      and PRC trigger times agree.
    - **Artifacts** — paths to `delta_t_hist.png` and
      `trigger_time_agreement.png`.
    - **Env used** — path of the shared env reused or created.

## Expect
- `~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ/report.md` exists
  and contains all five sections above with concrete numeric values
  (no TODO / placeholder text).
- `~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ/delta_t_hist.png`
  exists and shows both fitted_delta_t and prc_delta_t distributions
  in ns with the fitted peak annotated.
- `~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ/trigger_time_agreement.png`
  exists and shows the fitted-vs-PRC trigger time comparison.
- `~/phase2_MiniDAQ-no_scin_event_driven-GUI_DAQ/reference/phase2_MiniDAQ/`
  and `.../reference/SciFi/` directories exist (the cloned repos).
- The fitted_delta_t peak reported in `report.md` is a real numeric
  value located in the range 150 ns – 350 ns.
- `report.md` states an explicit integer `N_match` (count of events
  with `|fitted − PRC| < 50 ns`), the total event count, and the mean
  and std of `fitted − PRC` in ns.
- The bugs section either lists at least one concrete fix with
  file:line, or explicitly states "No bugs found" with a comparison
  summary against the reference decoder.
- Existing project files outside of the actually-buggy lines and the
  newly created `report.md` / PNGs / `reference/` directory are
  unchanged (no wholesale rewrites).
