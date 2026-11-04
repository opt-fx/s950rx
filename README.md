# S950rx

A browser editor and librarian for the Akai S900 and S950 samplers. It sends and fetches samples, edits them, slices breaks, builds programs and records through the sampler's own converters, over RS232 or MIDI. It runs in Chrome or Edge with no install, and the whole app is one HTML file.

**Status: release candidate.** It is built for the **stock ROMs V1.0A and V1.2A** and has been tested on a real S950 running V1.2A. The S900 shares the same exclusive protocol, so it should work, but it has not been tried on one. Other ROMs (V1.2B and custom builds) are not supported: the record automation and the display reading refuse to run on them.

## Quick start

1. Open the app's page, or download `index.html` and open it from disk, in **Chrome or Edge** (Safari and Firefox have no Web MIDI or Web Serial). Nothing else is needed.
2. Pick **Demo** and press **Connect** to try everything with a simulated sampler held in the page, or pick **RS232** or **MIDI** for a real one (see Setup and Connecting).
3. Use the four tabs: **Send samples**, **Edit sample**, **Edit break** and **Build a program**. **Sampler memory** on the left lists what the sampler holds.

## Setup

For RS232, which is what sample transfers and recording need:

- **A USB-to-serial adapter.** Tested with a Sabrent CB-DB9P (Prolific chipset).
- **A null-modem cable to the sampler's RS232 socket,** which is a DB25. Tested with a DB9 female to DB25 male null-modem cable (1.5 m) from the adapter's DB9 end.
- **Secure the plug.** The RS232 socket on the sampler is held to the chassis by screws, and the plug may not seat deeply enough over the pins with them in place. Replacing them with hex jackscrews (standoffs) lets you tighten the plug in, so it cannot work loose or fall out mid-transfer. Check the thread first; D-sub sockets usually take 4-40 UNC. This is a recommendation, not a known requirement.
- **The baud on the sampler.** Note the figure on its RS232 page and put the same in the app's **Baud** box. The sampler goes up to 100000 baud: for that speed, type **99999** on the sampler, then type **100000** in the app's Baud box.

**MIDI connected both ways.** A MIDI interface with one cable from the computer to the sampler's MIDI IN and one from its MIDI OUT back to the computer, with the sampler on **MIDI page 5, Control by** MIDI. The app needs both directions to read the sampler back, and it is also how the app switches the sampler over to RS232 (**Switch to RS232**), so you want it even if you mostly work over RS232.

## Connecting

| Mode | What it is for | Notes |
| --- | --- | --- |
| **RS232** | Everything, including sample transfers | Needs a USB-serial adapter (tested with a Prolific one). Set the **Baud** box to the figure the sampler is set to (up to 100000, which is 99999 on the sampler's page; see Setup). Hardware flow control is used from 50000 baud up. |
| **MIDI** | Parameter edits, program changes, audition, live display reading | Needs the MIDI ports chosen under **Sampler control**, with a cable both ways. Sample dumps do not pass through many MIDI paths (a sequencer in the way can drop them), so the app switches to RS232 for a sample send and hands back to MIDI afterwards. |
| **Demo** | Trying the app, testing | A simulated sampler with a few samples and two programs. |

The sampler powers up under MIDI control. **Switch to RS232** (under Sampler control) flips it over by SysEx: it sets the baud in the box, then opens the serial port. **Hand back to MIDI on disconnect** flips it back. Over RS232 the app cannot change the baud, because the sampler no longer listens on MIDI; if nothing comes back, the Activity line says which baud was tried. Connect over MIDI and press Switch to RS232 again to fix a mismatch.

The sampler must be on **MIDI page 5, Control by** MIDI or RS232 to match, and notes only sound while it is on MIDI control. **Omni** (a tick box under Sampler control) shows and sets the sampler's Omni setting.

**Test MIDI route** (under Sampler control, MIDI only) finds the largest program your MIDI path can carry. It sends test programs of 1, 2, 3, 4, 6, 8, 12, 16, 24, 32, 48 and 64 keygroups to the `AUDITION` slot, reads each back and compares it, and reports the biggest that arrived intact. The result is remembered for that pair of MIDI ports. After that, sending a larger program offers a detour over RS232 instead of failing. If nothing gets through, the app says to use RS232 for programs.

## The tabs

### Send samples

Drop WAV or AIFF files in and send them all to the sampler. Each is converted to the sampler's 12-bit format. The queue lets you set the name on the sampler (10 characters), stereo handling (mix or one side), normalise and the rate, and shows each file's length and loop. A name that is unique after the sampler's 10-character cut is left alone; files that would clash get `_01`, `_02` … in their last characters.

**Record through the sampler** plays queued files into the sampler's LINE input so the take picks up the machine's own converters. See Record below.

### Edit sample

Loads a sample from the sampler (✎ beside it in Sampler memory, or the **Sample** menu) or opens a WAV from your computer with **Open WAV…**. A ReCycle loop marker in a WAV is honoured: the sample is trimmed to the loop and any tail is dropped.

- **Markers:** drag the green start, red end and blue loop markers (snapping to zero crossings if ticked). Drag across the waveform to select; **Sel → start / end / loop** set a marker from the selection. The numeric fields do the same. A loop is the last *Loop length* words up to and including the end marker.
- **Parameters** (name, mode, start, end, loop start, loop length, root note, loudness) go to the sampler as you change them, or with **Send parameters**. They never touch the audio.
- **Audio tools:** trim to selection, crop to start/end, trim silence, reverse, normalise, remove DC, fades (with a curve choice), ±1 dB, **Match loop end**, **Loop crossfade**, **Resample** and **Timestretch**. Edited audio only reaches the sampler when you press **Send edited audio to sampler**, which replaces the sample of that name.
- **Back / Forward** walk one timeline of every change on the tab: audio, markers, slice settings and parameters. They keep your zoom.
- **Preview** plays in the browser (Space starts and stops it, wherever you are). **Play on sampler (hold)** works over MIDI: it writes a one-keygroup program called `AUDITION` pointing at the sample, selects it with a program change and plays held notes.

Suggested flow: connect over RS232 for the heavy work (load, send audio), then switch to MIDI for parameter tweaks with live audition.

### Edit break

The same editor and waveform as Edit sample, with the slicing tools on.

- **Slicing:** **Find hits** puts a cut just ahead of each drum hit (the **Sensitivity** slider re-runs the search live; at zero it finds nothing, so only pinned markers remain). Right-click the waveform to add or delete a marker, or double-click. The numbers in the table can be edited too.
- **Tempo:** say how many beats the loop is (or type its BPM). **Trim to exact length** cuts the play region to that many beats. **Fit to BPM** changes the tempo; the slider chooses how much of the change is a plain repitch (like a record speeding up) and how much a pitch-keeping stretch.
- **Moving markers:** drag the grip at the top of a marker. The slices on either side are stretched so the total length stays the same. Right-click a marker for **Quantise to** quarters, eighths or sixteenths; **Quantise all** does every unlocked marker at once (one step in Back / Forward). **Show grid** draws the grid.
- **Pin and Lock:** each row has two padlocks. **Pin** fixes the marker's position (Find hits, sensitivity, Quantise all and dragging leave it alone); a pinned marker is orange. **Lock** fixes the row's settings (Set, Randomise and Reset all leave them alone); a locked slice is tinted blue. Shift-click sets every row.
- **Timestretch per slice:** a stretch percentage, **D-time**, **Auto D** and **Echo** (milliseconds or a note value at the loop's BPM) for each slice, with dice buttons to randomise. See Timestretch below.
- **Scratchpad** keeps slice settings you like (+ Save) to come back to.
- **Output:** **Send slices** sends each slice as a one-shot sample (`NOOKIE_01`, `NOOKIE_02`, …); **Send slices & map as drum kit** also puts them on consecutive keys from the **Start key**, retuned so stretched slices keep their original length. **Render to one sample** bakes the sliced and stretched break into one sample. **Export WAV**, **Export slices** (a zip) and **Export pattern (.mid)** write files; the pattern plays the slices in their original order at the loop's tempo.
- Loading a new file clears the slicer back to its defaults.

### Build a program

The S950 has no multi mode, so one program can mix a drum kit and pitched sounds.

- **New program** empties the builder. Load an existing program into it with ✎ in Sampler memory. Rename it and send, and it is renamed in its own slot, or tick **Send as a new program instead** to keep the old one.
- **Drum kit:** choose a start key and tick samples; each gets its own key at constant pitch, in the order they sit on the sampler. **Pitched zone:** one sample over a key range that tracks the keyboard from its own root note. **Auto-map** spreads every sample across the keyboard.
- The keygroup table has key range, sample, transpose and fine tune (1/16 semitone steps), filter, loudness, an A D S R envelope, mod, AT, positional crossfade, **1S** (one-shot) and the output. **Show all settings** opens the rest of the sampler's keygroup parameters, including the LFO. In the **Out** column, shift-click applies a choice to every row. A keyboard strip shows what covers which keys. Warnings appear for shared keys, missing samples and the 64-keygroup limit (which blocks sending).
- **Fit to tempo** turns a source and target BPM into a Tune value (semitones = 12 × log2(target ÷ source)); pitch moves with speed, as on the sampler. The readout shows the tempo the quantised tune really gives.
- **LFO Rate from tempo** (under Show all settings) sets the LFO Rate so one cycle lasts a chosen note length at a BPM. On a stock V1.2A the LFO runs from about 1.8 Hz (Rate 0) to 10.6 Hz (Rate 99); it warns when a tempo is outside that, and logs the two nearest Rates.
- **MIDI prog#** is shown from 1, as the unit shows it. The sampler picks programs by this number, so two programs with the same one cannot be told apart: the app shows clashes in amber in Sampler memory and refuses to send one. Hover a program in Sampler memory and press ◉ to make it the active program (a MIDI program change).
- **Replace TONE PRGRM** writes over the factory program in slot 1 so the front panel's PLAY page shows your program's name.
- If the program on the sampler has changed since you loaded it (on the front panel, say), sending asks whether to overwrite it or load the sampler's version first. A program too big for the MIDI route is offered a detour over RS232.
- **Save to file** and **Open file** keep the layout on your computer as `.s950prog.json`.

## Sampler memory and the display

**Sampler memory** lists programs (numbered from 1, with their MIDI prog# in brackets) and samples (numbered from 0, as the sampler counts them). Hover a row for its actions: ✎ edit, ⭳ fetch as WAV, ◉ select. **Back up all** (RS232 only, as sample dumps cannot pass through MIDI) saves every sample as a WAV and every program as a `.s950prog.json` file.

The LCD at the top is the app's own readout. **READ UNIT (HOLD)** shows what the sampler's own two lines are showing while you hold the button down, and it changes to UNIT DISPLAY when it is lit. It reads one display per press and never polls. See Service ops below. Shift-click the display to change its colour; the choice is remembered.

## Record through the sampler

Part of the character of a vintage sampler is its converters. **Record…** on a queued file plays it from the computer into the sampler's LINE input, so the take is made by the sampler itself. Cable the computer's audio output to the LINE input and pick the output in the panel.

On a V1.0A or V1.2A the app drives the record pages. It writes a placeholder name, types the bandwidth and recording time (worked out from the file), and waits at the level page. You audition the file in a loop, set REC LEVEL and the threshold, press ENT on the sampler and let the app play it. **Check recording** then finds the new sample, checks its rate and length against the plan, gives it its real name and sets its pitch to C3 so it plays at the recorded speed.

**Record all through the sampler…** repeats this for every queued file. The level is set once on the first take, or before every take if you tick *Set the level for every take*. For each take the only manual step is pressing ENT during the countdown (8 s by default): the stock sampler does not take ENT from the app, and the level page does not answer MIDI. Files already on the sampler are skipped, so Start again carries on after a failed take. Samples made this way end in `*` (`AMEN*`), which can be switched off. Sending a program puts the sampler back on PLAY if it was left on the record pages.

## Timestretch (the sampler's own effect, in the app)

The metallic stretch of the sampler's EDIT SAMPLE page 14 (the sound of classic jungle) is reproduced in the app, so it works on any sample you can load, with no sampler involved, and gives S900 owners on the stock OS an effect their unit does not have. It is separate from **Fit to BPM**, which is the clean tempo fit.

It was worked out from the sampler's own output: a break, a click train and a held tone were stretched on a real S950 (V1.2A), fetched back and compared word for word with the app's result. Nine runs agree to the 12-bit limit, every output length matches exactly, and by ear the two are identical.

How it works: the sound is played in segments copied unchanged from the source. At every join the source steps back D words (the **D-time**, 5 to 9999) so the last D words are heard again; that repeat is the metallic sound (a small D gives a ring-modulator buzz, a large D gives echoes). A segment is T = D × p / (p − 100) words long (p is the percentage), which makes the whole run at p%; under 100% the same thing runs forwards. Neighbouring segments are blended over one segment length: equal-power for **Poly** (the sampler's default), a straight linear fade for **Mono**. **Auto D** sets D to the sound's pitch period in words. Stretched versions are named like `AMEN#200`.

**Timed echoes** are an addition the sampler does not have. Instead of typing a D-time in words, give a slice an echo time, either in milliseconds or as a note value at the loop's BPM (`1/16`, `1/32T` for a triplet, `1/64.` for a dotted note), and the app works out the D-time for it, so the metallic repeats stay in time with the music. The dice button gives each stretched slice its own random echo, from note values between two you pick (so they still sit in time) or from a range in milliseconds.

What is not exact: the rule for how many joins fit at the end is fitted to the measurements, so a length could differ by one join (D words) from the sampler's; its 12-bit rounding is not copied; and percentages and D-times at the extremes have not been run on a sampler.

## Service ops (stock ROM)

The stock V1.0A / V1.2A ROM answers three service ops outside the documented message set: PEEK (`0E`, reads RAM), POKE (`0C`, writes RAM) and CALL (`0D`, never used here). The app uses PEEK and POKE to read which section and record page the sampler is on, to press panel keys by writing the key buffer, and to type values into the record pages. Addresses depend on the ROM and are detected from the sampler. They work over MIDI and over RS232. Only value fields are written, and a power cycle recovers from any mistake.

Each section's page text sits in a RAM table, with the page being shown marked by a zero byte, so the display can be read in one request without polling. Continuous polling is not recommended: it makes the MIDI lamp flash constantly and the sampler drops the occasional request when it switches section.

- **PLAY:** the program list is a 40-column window onto two long strips in RAM; the knob's click count is the window's offset, so the readout scrolls as the sampler does. With a single program the strips hold a different layout, which the app also reads.
- **MIDI, DISK, EDIT PROGRAM, EDIT SAMPLE, UTILITY:** each has a page-text table in RAM (80 characters per page, live values included).
- **RECORD:** each page's title text is laid out as on the sampler. The level page and the replay screen do not answer MIDI, so they are drawn as photographed (the trigger marker is fixed, not live).
- **MASTER TUNE** has no lamp, so the sampler still reports PLAY; the app reads the tuning screen directly.

The LCD uses the sampler's own character shapes (the HD44780 A00 set, plus the page arrow copied from a close-up).

## Confirmed on real hardware

All on an S950 with V1.2A, over RS232 (a Prolific USB adapter, up to 100000 baud with hardware flow control) and over MIDI:

- Catalogue, sample parameters, program and overall-settings reads; sending a WAV and fetching it back with audio intact; sending a long run of slices.
- Switching between MIDI and RS232 from the app, including blind from the settings saved at the last RS232 session when the MIDI path returns nothing.
- The sample editor: markers sent live, trimming and sending back at the same index, loops (verified against a real sample), Match loop end, Loop crossfade, and audition.
- Program edits: sending, renaming in place, replacing TONE PRGRM, MIDI prog# changes and program changes, retuned keygroups and drum-kit timing, the one-shot flag, shift-click spreading, and the LFO tempo helper (in sync at 1/16, 1/8, 1/4 and 1/8 triplet; about 2% off where the Rate steps allow).
- Fractional keygroup tune: 440 Hz sines at Tune 0 read 440.5 Hz and at +0.9375 read 464.8 Hz (predicted 464.5), −12 read 220.2 Hz.
- Reading the display on every screen, record automation (a single take over MIDI and a multi-file sequence over RS232 at 100000 baud), and Timestretch against the sampler's own output.
- Sample dumps and larger programs (9 keygroups failed, 2 were fine, found with Test MIDI route) do not get through some MIDI paths with a sequencer in the way; the app sends those over RS232 instead.

## Not tested or not done

- **S900:** never run on one. The protocol is shared, so it should work.
- **ROMs other than stock V1.0A / V1.2A:** V1.2B is a different build and is not supported.
- **Export pattern (.mid)** in a sequencer or DAW.
- The record automation's "any key" start, which might remove the ENT step, and Timestretch at the extremes of its ranges.
- Saving disk images (`.img`, `.hfe`) and reading them back are not built.

## Credits and licence

S950rx is a browser port and extension of **S950rs** (v2.4, [github.com/dxzl/akai-s950](https://github.com/dxzl/akai-s950)) by **Scott Swift (dxzl)**, who wrote the original concept and code. The name keeps that lineage: rs became rx. The protocol code is derived from S950rs, with help from an S900/S950 disk-format study. Scott has confirmed that he is happy for S950rx to be a port of his work, and regards S950rs as open source.

The Fit to tempo calculation is based on OPT's pitch/tempo calculator. Released under the **GPL-3**, as S950rs is (see `LICENSE`). Ported, extended and maintained by OPT.
