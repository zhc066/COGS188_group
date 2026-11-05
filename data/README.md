# Data sources and preparation

Detailed notes behind the Data section of the proposal. Real data only calibrate the instance generator; the solvers never see real UCSD records.

## Sources

| Source | Records | Fields used |
|---|---|---|
| [UCSD Historical Enrollment Data](https://github.com/UCSD-Historical-Enrollment-Data/UCSDHistEnrollData) (community-maintained, MIT license) | Enrollment snapshots per section and collection time; per-term schedule files in `data/schedules/*.tsv` (e.g., `FA25.tsv`) | Snapshots: `subj_course_id`, `sec_code`, `sec_id`, `enrolled_ct` / `enrolled`, `total`. Schedules: `subj_course_id`, `sec_code`, `instructor`, `total_seats`, `meetings` (each meeting is `<code>,<days>,<time>,<room>`, e.g., `LE,TuTh,11:00 - 12:20,PETER 110`) |
| [UCSD Classroom Attributes Report](https://facilities.ucsd.edu/NA/ClassroomAttributes/Default.htm?view=ClassroomAttributes) (Facilities Information System; Excel export available) | One record per Registrar-controlled classroom | Building, room, registrar code, capacity, room type, used / Registrar-controlled indicators |

Record the quarter, file names, access dates, record counts, and reuse terms before freezing the generator.

## Enrollment and schedule preparation

- Use one well-documented fall, winter, or spring quarter and a declared set of departments. Avoid Spring 2022, whose documented schema lacks enrollment counts.
- Use one observation per term and section: the latest snapshot on or before a common reference date near the start of instruction. Report its age and missing coverage. Repeated collection timestamps are not independent classes.
- Keep lecture meetings only (`LE`); drop discussions (`DI`), finals (`FI`), and other meeting codes.
- Each schedule row is a section (e.g., `COGS 100 A01`, `A02`, `A03`) and sections of one lecture repeat the same `LE` meeting. Merge rows with the same course and identical `LE` meetings into one lecture; its class size is the combined enrolled count of those sections, so discussion sections are not counted as separate courses.
- Meetings per week come from the lecture day pattern (e.g., `TuTh` = 2, `MWF` = 3). Keep lectures that meet two or three days per week; drop one-day, TBA/date-only, and four-plus-day patterns. Clock times and durations are not used.
- Fall 2025 reference counts (`FA25.tsv`): 6,502 section rows, 1,519 distinct lectures, of which 789 meet two days and 377 three days (77%); lecture seat totals have median 50, 90th percentile 196, 99th percentile 384, and 8 entries at or above `9999`; 126 lectures list `Staff`.
- Inspect missing values, zero enrollments, unusually large limits such as `9999`, and enrollment above the reported limit; document every exclusion.
- Use enrolled counts for demand; do not infer enrollment from `available`. `total` / `total_seats` is a registration limit, not a physical room capacity.
- Do not merge `Staff` entries into one shared instructor; handle multiple-instructor records explicitly before deriving teaching loads.

## Classroom preparation

- Normalize room identifiers and remove duplicates; keep active, relevant teaching rooms and drop rooms listed with capacity 0.
- Reference counts (accessed November 2026): 141 Registrar-controlled rooms (97 classrooms, 39 lecture halls, 4 assembly rooms, 1 class laboratory); capacity median 50, 90th percentile 245, maximum 500.
- Drop lectures larger than the largest room. When sampling an instance's room inventory, resample until every course fits at least one room and report the resampling rate.
- The inventory may describe a different date from the enrollment quarter, so the combined data describe hypothetical scenarios.
- The inventory does not establish room availability; experiments assume every sampled room is available in every period. The same assumption applies to every method.

## Calibration checks

- Sample room capacities before constructing the hidden feasible timetable; never enlarge rooms afterward to fit assigned courses.
- Check that every course has a suitable room and that a complete feasible timetable can be built.
- Compare source and generated distributions of enrollment and capacity, suitable-room counts, and generation rejection rates to see whether feasibility filtering shifts the sampled distributions.

## Synthetic components

- **Cohorts.** The sources have no student-level co-enrollment. A synthetic cohort is a group of courses whose meetings cannot overlap; courses may belong to more than one cohort. Cohort counts, group sizes, membership rules, and overlap settings are chosen on development instances, checked against the 30-period weekly load, and frozen before held-out evaluation. They are experimental conflict structures, not measured UCSD cohorts.
- **Instructor availability.** A published teaching assignment does not reveal all times an instructor is available, so availability is generated under documented rules.
- **Undesirable periods.** Synthetic assumption.

## Time grid

The model uses 30 ordered, equal abstract periods (5 days × 6). UCSD's [standard teaching times](https://blink.ucsd.edu/instructors/courses/schedule-of-classes/times.html) mix 50-minute MWF and 80-minute TuTh meetings; we keep two or three one-period meetings per course on distinct days and do not map real clock times onto periods. Idle-gap penalties therefore count abstract periods, not minutes. Variable-duration meetings would need a revised grid and are out of scope.
