# Circular task analysis

Final project of the course *Python, Git and RStudio* (Master IEAP, Université de Montpellier).

**Authors:** Gaia GASSIOLE, Zoé SAUGE, Jeanne LE ROUX — November 2026

---

## 1. Context

The scientific question behind this project is: *can a circular steering task quantify sensorimotor integration in persons with stroke?* (Fauviaux et al., 2021).

In the task, the participant moves a cursor with the mouse along a circular track (a ring between two circles) as fast and as accurately as possible, during several records of 20 s.
The task is run with **MouseReMoCo**, which records the position of the cursor and computes a table of results at the end of the session.

## 2. Goals

1. Reproduce the figures of the assignment from the files `001MoDe_R1.csv` and `001MoDe_R1.marker.csv`.
2. Recompute the table of results (nLaps, Re, Te, error, MT/lap, IDe/lap, Be, IPe) with our own code and compare it to the table of MouseReMoCo.
3. Run the same code on a recording that we made ourselves.

Only `numpy`, `matplotlib` (with `Rectangle` and `Circle` from `matplotlib.patches`) and `os` are used, as asked in the assignment.

## 3. How to run

1. Clone or download the repository.
2. Open the folder in VS Code, with a Python environment that contains `numpy`, `matplotlib` and `ipympl` (for the interactive figures, `%matplotlib widget`).
3. Open `main.ipynb` and click **Run All**.

`main.ipynb` runs the whole analysis on the reference recording, then on our own recording.
Each notebook in `notebooks/` can also be run on its own.

To analyse another recording, put its two files in `data/` and set its name (without `.csv`) at the beginning of the notebook:

```python
os.environ["MOUSE_FILE"] = "002SaZo"
```

If `MOUSE_FILE` is not set, the reference recording `001MoDe_R1` is used.

## 4. Repository structure

```
circular-task-analysis/
├── main.ipynb                  # runs the whole analysis, with a summary of each part
├── notebooks/
│   ├── 01_reading.ipynb        # read the files and split the session into records
│   ├── 02_graphics.ipynb       # figures of the movement
│   ├── 03_computing.ipynb      # statistics of each record
│   ├── 04_comparison.ipynb     # comparison with the table of MouseReMoCo
│   └── 05_owndata.ipynb        # analysis of our own recording, comparison of the two hands
├── data/
│   ├── 001MoDe_R1.csv          # reference recording (given with the assignment)
│   ├── 001MoDe_R1.marker.csv
│   ├── 002SaZo.csv             # our own recording
│   ├── 002SaZo.marker.csv
│   └── readme.md               # description of the data files
├── readme.md
└── LICENSE
```

## 5. Data

MouseReMoCo writes two files for each session:

- **`001MoDe_R1.csv` and `002SaZo.csv`**: the parameters of the task (first line, `name value` pairs separated by `;`), the date, an empty line, then one line per sample: `timestamp` (ms), `mouseX`, `mouseY` (pixels, y pointing down) and `mouseInTarget` (1 = inside the target, 0 = outside). 
A line `0,0,0,0` is written at the end of each record: these lines are not real samples and are removed.
- **`001MoDe_R1.marker.csv` and `002SaZo.marker.csv`**: the same parameters, the events of the session (`DoRecord` = start of a record, `DoPause` = end of a record) and the table of results computed by MouseReMoCo.

| Recording | MouseReMoCo | Records | Task radius | Task tolerance | Notes |
|---|---|---|---|---|---|
| `001MoDe_R1` | 1.2.0rc5 | 5 × 20 s | 209.5 px | 47 px | reference recording; the 6th record is empty (window closed during the pause) |
| `002SaZo` | 1.3.0 | 6 × 20 s | 199.5 px | 47 px | our recording, done with a touchpad: records 0–2 right hand, 3–5 left hand |

## 6. Methods

### 6.1 Reading the data (`01_reading.ipynb`)
- The parameters are stored in a dictionary (`params["centerX"]`, `params["taskRadius"]`, ...).
- The `0,0,0,0` lines are removed. Time is converted to seconds, with t = 0 at the first sample.
- Each record is the part of the data between a `DoRecord` and the next `DoPause` marker.

### 6.2 Figures (`02_graphics.ipynb`)
- Coordinates are centered on the circle, with the y axis pointing up: `x_c = x - centerX`, `y_c = centerY - y`.
- The center of the cursor is inside the target when its distance to the center is between `internalRadius + cursorRadius` and `externalRadius - cursorRadius - borderRadius` (186 and 233 px for the reference recording).
- Figures: x and y over the whole session, trajectory of the whole session, trajectory of each record (inside points in green, outside points in red), distance to the center over time, and x, y and mouseInTarget of each record over time.
- Each figure is named with the name of the recording, so that the two analyses can run in the same notebook without mixing their figures.

### 6.3 Statistics (`03_computing.ipynb`)
As in MouseReMoCo, each record is kept from the **first entry into the target**. Then, with r the distance to the center and θ the angle of the cursor:

| Variable | Formula | Meaning |
|---|---|---|
| nLaps | (θ_end − θ_start) / 2π, with `np.unwrap(θ)` | number of laps (negative = clockwise, with y pointing up) |
| Re | mean(r) | effective radius: radius really followed |
| Te | 4.133 × SD(r) | effective tolerance: width of the path really used |
| error | 100 × angle travelled outside the target / total angle travelled | % of the lap outside the target |
| MT/lap | duration / \|nLaps\| | movement time per lap |
| IDe/lap | 2π × Re / Te | effective index of difficulty of one lap (steering law) |
| Be | Te / taskTolerance | effective tolerance compared to the one asked |
| IPe | IDe/lap / MT/lap | effective index of performance |

With the task radius and tolerance, 2π × 209.5 / 47 = 28.01 bit/lap: this is the `indexOfDifficulty` given in the header, which confirms the formula of IDe/lap.

### 6.4 Comparison (`04_comparison.ipynb`)
The table at the end of the marker file is read automatically and compared to our values, variable by variable.

## 7. Results

### 7.1 Reference recording (`001MoDe_R1`)

| Record | nLaps | Re (px) | Te (px) | error (%) | MT/lap (s) | IDe/lap (bit) | Be | IPe (bit/s) |
|---|---|---|---|---|---|---|---|---|
| 0 | −10.84 | 213.86 | 91.79 | 3.90 | 1.84 | 14.64 | 1.95 | 7.94 |
| 1 | −11.97 | 209.85 | 34.45 | 0.74 | 1.59 | 38.27 | 0.73 | 24.04 |
| 2 | −13.26 | 209.43 | 42.42 | 2.81 | 1.45 | 31.02 | 0.90 | 21.40 |
| 3 | −12.83 | 209.46 | 37.14 | 0.13 | 1.49 | 35.43 | 0.79 | 23.72 |
| 4 | −14.74 | 210.89 | 43.88 | 2.84 | 1.30 | 30.20 | 0.93 | 23.20 |

- All values are the same as the MouseReMoCo table, with a difference of at most 0.01 (rounding).
- The number of laps increases overall over the session (from 10.84 to 14.74 laps in 20 s), but not at every record.
- Record 0 has a much larger Te (91.79 px) because of a loop at its beginning: its IDe/lap and IPe are low.

### 7.2 Our recording (`002SaZo`)

| Record | Hand | nLaps | Re (px) | Te (px) | error (%) | MT/lap (s) | IDe/lap (bit) | Be | IPe (bit/s) |
|---|---|---|---|---|---|---|---|---|---|
| 0 | right | −5.99 | 197.27 | 42.79 | 2.22 | 3.24 | 28.97 | 0.91 | 8.93 |
| 1 | right | −6.00 | 191.70 | 52.59 | 8.84 | 3.23 | 22.90 | 1.12 | 7.10 |
| 2 | right | −7.73 | 189.99 | 43.88 | 4.86 | 2.54 | 27.21 | 0.93 | 10.70 |
| 3 | left | −6.03 | 197.08 | 52.71 | 6.32 | 3.25 | 23.49 | 1.12 | 7.22 |
| 4 | left | −6.43 | 190.00 | 61.45 | 16.16 | 3.02 | 19.43 | 1.31 | 6.43 |
| 5 | left | −8.02 | 195.77 | 61.92 | 10.47 | 2.43 | 19.87 | 1.32 | 8.19 |

- The values match the MouseReMoCo table with a difference of at most 0.03.
- With the left hand, the speed is similar (MT/lap 2.90 vs 3.00 s), but the movement is less accurate: Te is larger (58.7 vs 46.4 px on average), the error is about twice as high (11.0 vs 5.3 %) and IPe is lower (7.3 vs 8.9 bit/s).

## 8. Critical points and how we solved them

- **Formula of the error**: the percentage of samples outside the target did not give the values of MouseReMoCo (5.39 % instead of 3.90 % for record 0). The error is the percentage of the **angle travelled** outside the target: the cursor slows down when it leaves the target, so counting samples overestimates the error.
- **Assembling the notebooks**: when `main.ipynb` ran all the notebooks, the reading part was run twice and `plt.close("all")` closed the figures already displayed. Each notebook now runs the previous one only if its variables do not exist yet, and each figure is closed and named individually.
- **Another version of MouseReMoCo**: our recording (version 1.3.0) has other parameters and 6 records instead of 5. The file is chosen with `MOUSE_FILE` and the figures adapt to the number of records, so the same code works for both files.
- **Sign of nLaps**: it depends on the direction of the y axis. We chose y pointing up (as in a usual graph): the cursor turns clockwise, so nLaps is negative, as in the MouseReMoCo table.

## 9. Workflow

The project was split into 12 tasks, taken in turn by Jeanne, Gaia and Zoé, so that each of us worked on reading, graphics, computing and finalisation.
Each task had its own branch, created from an up-to-date `main`, and ended with a pull request reviewed by the other members. Jeanne (manager) merged the pull requests.

| # | Task | Who | Branch |
|---|---|---|---|
| 1 | Read the header | Jeanne | `01-read-header` |
| 2 | Read the data and the markers | Gaia | `02-read-data-markers` |
| 3 | Split the session into records | Zoé | `01_reading` |
| 4 | Overview figures | Jeanne | `04-plot-overview` |
| 5 | Trajectory of each record | Gaia | `05-plot-trajectory` |
| 6 | r, x, y and mouseInTarget over time | Zoé | `06-plot-radius` |
| 7 | nLaps | Jeanne | `07-compute-nlaps` |
| 8 | Re, Te and error | Gaia | `08-re-te-error` |
| 9 | MT/lap, IDe/lap, Be and IPe | Zoé | `09-mt-ide-be-ipe` |
| 10 | Assemble main and compare to MouseReMoCo | Jeanne | `10-assemble-main` |
| 11 | Run the code on our own recording | Gaia | `11-run-own-code` |
| 12 | Final cleanup: readme, removal of the template files, typos | Jeanne | `12-final-cleanup` |
| 13 | 1-page report (Moodle) | Zoé | — |

## 10. Limitations

- Only one participant per recording: the statistics are descriptive and no general conclusion can be drawn.
- Our recording was done with a touchpad instead of a mouse, which explains some discontinuous trajectories.
- With only three records per hand, the comparison of the two hands is only indicative.

## 11. References

- Fauviaux, T., et al. (2021). *Can a circular steering task quantify sensorimotor integration in persons with stroke?* Conference poster. https://hal.umontpellier.fr/hal-03834500 (context of the task, boxplot of IPe)
- Chapuis, O. *Pointage* (course on Fitts' law): https://perso.lisn.upsaclay.fr/chapuis/ihm-polytech/pointage.pdf (Re and Te)
- Accot, J., & Zhai, S. (1997). Beyond Fitts' law: models for trajectory-based HCI tasks. *Proceedings of CHI '97*, 295–302. (steering law, IDe/lap)

Software: MouseReMoCo (Denis Mottet), used to record the task and to compute the reference table.

## License

GNU General Public License v3.0 (see `LICENSE`).