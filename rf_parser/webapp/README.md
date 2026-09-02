# Pulseq Single-Channel Sequence Builder

A self-contained, dependency-free single-page web app for generating single-RF-channel
Pulseq sequence data from MRI parameters — no Python/Node install required.

Open `index.html` directly in a browser (or serve the folder statically).

## What it does

Enter parameters for one of three sequence types:

- **Gradient Echo (GRE)** — TR, single excitation pulse
- **Spin Echo (SE)** — TR, TE, excitation + refocusing pulse
- **Turbo / Fast Spin Echo (TSE/FSE)** — TR, echo spacing (ESP), echo train length (ETL),
  excitation + one shared refocusing pulse played `ETL` times

For each pulse you can set flip angle, duration, RF pulse shape (sinc with configurable
time-bandwidth product and Hann apodization, or a hard/block pulse), phase offset, sampling
points per pulse (`N`), and DAC bit depth. RF pulse amplitude (Hz) is computed from the
flip angle using the standard small-tip-angle integral
(`amplitude = flip_angle / (2π ∫ shape(t) dt)`); this is an approximation, most exact for
excitation pulses, and is not a full Bloch simulation — treat generated 180° amplitudes as
indicative starting points.

The app then produces, live:

1. **A pulse-sequence diagram** — canvas rendering of the RF envelope over one TR, with
   sidelobe polarity, TE/echo markers, and an optional zoom to the active RF window (TR is
   usually much longer than the RF pulse train).
2. **A single-channel JSON file** matching [`../schema.md`](../schema.md) /
   [`../schema.json`](../schema.json) (the `SingleChannelSequence` variant) — waveform
   magnitude/phase/two's-complement arrays plus a gap/pulse timeline.
3. **A Pulseq `.seq` file** — RF-only blocks (no gradients/ADC), readable by
   [`../rf_waveform_parser.py`](../rf_waveform_parser.py). Each unique pulse shape is written
   as verbatim (uncompressed) magnitude/phase/time shapes, so it round-trips exactly back
   through the parser.

## Verifying round-trips

The generated `.seq` output has been checked against `rf_waveform_parser.py` (peak RF
amplitude and normalized magnitude match exactly) and the generated JSON has been validated
against `schema.json` for all three sequence types.

```sh
python3 rf_parser/rf_waveform_parser.py my_sequence.seq -o my_sequence_reparsed.json
```

## Notes / limitations

- Single channel only (no `gain`/`phase` per-event fields — those apply to the
  multi-channel schema variant produced by `multichannel_json_parser.py`).
- One TR is generated (`tr_index: 0`); this is a parameter-driven waveform generator, not an
  extraction from a real acquired `.seq` file with multiple TRs.
- Amplitude scaling and phase offset are still present where the schema calls for them: each
  pulse's peak amplitude (`rf_amplitude_hz`) is the flip-angle-derived scale factor, and each
  pulse's `phase_rad` array carries the requested phase shift (e.g. the default 90°
  excitation/refocusing offset for the CPMG condition).
