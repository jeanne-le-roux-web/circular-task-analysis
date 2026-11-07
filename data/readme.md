# Data

This folder contains the recordings of the circular task made with MouseReMoCo.
Each recording has two files with the same name: `<name>.csv` (movement of the cursor) and `<name>.marker.csv` (events and results).

## Recordings

| | `001MoDe_R1` | `002SaZo` |
|---|---|---|
| Origin | reference recording, given with the assignment | our own recording |
| Date | 26 March 2021 | 27 October 2026 |
| MouseReMoCo version | 1.2.0rc5 | 1.3.0 |
| Device | mouse | touchpad |
| Records | 5 of 20 s (the 6th record is empty: the window was closed during the pause) | 6 of 20 s (records 0–2: right hand, records 3–5: left hand) |
| Samples (after removing the `0,0,0,0` lines) | 19588 | 17141 |
| Screen (width × height) | 1104 × 661 px | 1264 × 641 px |
| Center of the circle (centerX, centerY) | (552, 330) px | (632, 320) px |
| Internal / external radius | 170 / 250 px | 160 / 240 px |
| Cursor radius / border radius | 16 / 1 px | 16 / 1 px |
| Task radius (taskRadius) | 209.5 px | 199.5 px |
| Task tolerance (taskTolerance) | 47 px | 47 px |
| Index of difficulty (indexOfDifficulty) | 28.01 bit/lap | 26.67 bit/lap |

## Content of the files

### `<name>.csv`: movement of the cursor
1. **Line 1**: the parameters of the task, as `name value` pairs separated by `;` (e.g. `screenWidth 1104;screenHeight 661;...`).
2. **Line 2**: the date and time of the session.
3. **Line 3**: empty.
4. **Line 4**: the names of the columns: `timestamp,mouseX,mouseY,mouseInTarget`.
5. **Next lines**: one line per sample:
   - `timestamp`: time of the sample, in milliseconds since 1 January 1970 (Unix time);
   - `mouseX`, `mouseY`: position of the cursor on the screen, in pixels (origin at the top left corner, y pointing down);
   - `mouseInTarget`: 1 when the cursor is inside the target, 0 otherwise.

A line `0,0,0,0` is written at the end of each record. These lines are not real samples: they are removed when the data are read.

### `<name>.marker.csv`: events and results
1. **Line 1**: the same parameters as in the `.csv` file.
2. **Line 2**: the date and time of the session.
3. **Events**: one line per event, with its timestamp (ms), for example:
   - `DoCycleChange:DoRecord`: start of a record;
   - `DoCycleChange:DoPause`: end of a record (start of the pause).
4. **Table of results** computed by MouseReMoCo: one line `Theory` (values expected by the task) and one line per record (`Rec001`, `Rec002`, ...) with nLaps, Re, Te, error, MT/lap, IDe/lap, Be and IPe.

## Notes

- The target zone for the **center** of the cursor is between `internalRadius + cursorRadius` and `externalRadius - cursorRadius - borderRadius` (186 and 233 px for `001MoDe_R1`, 176 and 223 px for `002SaZo`).
- In version 1.3.0, a few samples are also recorded during the pauses: they are not used, because each record is cut between its `DoRecord` and `DoPause` markers.
- To analyse a new recording, put its two files in this folder and set its name (without `.csv`) with `os.environ["MOUSE_FILE"] = "<name>"` (see `notebooks/05_owndata.ipynb`).