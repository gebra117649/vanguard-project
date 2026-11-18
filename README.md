# Vanguard Digital Experience Analysis

## Table of Contents

- [Team Members](#team-members)
- [Business Overview](#business-overview)
- [Business Questions](#business-questions)
- [Dataset](#dataset)
- [Data Preparation](#data-preparation)
- [Python Analysis](#python-analysis)
- [Key Findings](#key-findings)
- [Business Recommendations](#business-recommendations)
- [Presentation](#presentation)
- [Repository Structure](#repository-structure)
- [How to Run the Project](#how-to-run-the-project)

## Team Members

**Vanguard CX Analytics Team**

- Telma Guerreiro
- Erzsebet Fuzes
- Marta Atencia

## Business Overview

Vanguard is a US-based investment management company. Its Customer Experience (CX) team wanted to understand whether an updated website would make an existing online process easier for clients to complete. The redesign combined a more intuitive interface with prompts shown during the process to help clients understand what to do next.

Vanguard ran an A/B test from **March 15 to June 20, 2017**. Clients were randomly assigned to one of two groups. **Control** used the original website experience, and **Test** used the redesigned interface. Both groups completed the same five-step process:

`start → step_1 → step_2 → step_3 → confirm`

As the CX Analytics team, our goal is to help Vanguard decide **whether the redesign should replace the original experience**. The project brief asks whether clients complete the process more often, how long it takes them, how they navigate the steps, and where they still get stuck. The event logs show each recorded action, so we reconstruct sessions before measuring these outcomes.

We also look at who experiences difficulty and when. The age analysis helps us identify clients who may benefit from clearer guidance, while the time-of-week analysis helps us understand whether the results are consistent across market hours, off-hours, and weekends.

## Business Questions

### H1 — Did the redesign improve completion and reduce friction?

- Did the redesigned website increase the session completion rate by at least 5%
- Did the redesign reduce the proportion of high-friction journeys?
- At which steps were clients more or less likely to skip, repeat, or move backward in Test compared with Control?
- Which steps should the CX team investigate before a wider rollout?

### H2 — How does age relate to client success?

- Among clients who succeeded at least once, did clients under 48 need fewer recorded visits to reach their first success than clients aged 48 or above?
- How likely was each age group to record at least one successful visit during the experiment?
- Did both age groups see higher success rates in Test? Was the observed age gap narrower or wider on the redesigned interface?

### H3 — Does performance vary by time of week?

- How is session traffic distributed across weekday market hours, weekday off-hours, and weekends?
- How do completion, session duration, and navigation friction compare between Test and Control within each period?
- When did the redesign show its largest observed completion gain, and what could that mean for the timing of a phased rollout?

## Dataset

We used the Vanguard experiment files supplied with the Ironhack project brief.

| Source file | Contents |
|---|---|
| `df_final_web_data_pt_1.txt` and `df_final_web_data_pt_2.txt` | Web events with client, visitor, and visit identifiers, process steps, and timestamps |
| `df_final_experiment_clients.txt` | Test and Control assignments by client |
| `df_final_demo.txt` | Client demographics, tenure, account count, account balance, calls, and logons |

The experiment assigned groups at the **client level** using `client_id`. We identified sessions using `client_id` and `visit_id` together. `visitor_id` identifies a browser or device associated with a client. A client can use several devices and contribute multiple sessions.

## Data Preparation

1. **Combine the web logs.** Stack the two web-event files and join the experiment assignments on `client_id`. Join those assignments to the client profiles for the demographic analysis.
2. **Clean and export the inputs.** Remove duplicate web events and records with missing fields. Remove client profile rows without an experiment assignment or client ID. Standardise column names and export `cleaned_web_data.csv` and `cleaned_client_data.csv`.
3. **Build the base session table.** Group the cleaned events by client and visit, count the recorded events per visit, and export one row per session to `sessions_data.csv`.
4. **Reconstruct the journeys.** Sort events chronologically within each visit. Record the start and end steps, measure elapsed time, and count skips, repeats, and backward moves. Export the enriched table as `sessions_data_master.csv`.
5. **Prepare the H1 export.** Add the H1 completion and friction measures to the session data and export `h1_sessions.csv` for Tableau.

## Python Analysis

| Notebook | Role |
|---|---|
| `data_cleanup.ipynb` | Combines and cleans the source files |
| `Cleaned_session_data.ipynb` | Creates the base session table |
| `demo_data_analysis.ipynb` | Profiles clients and compares Test and Control demographics, account characteristics, and prior engagement |
| `Session_data_analysis_day_1.ipynb` | Explores event timing, step activity, and where recorded sessions end |
| `journey_status.ipynb` | Explores confirmation events and navigation order |
| `master_session_data.ipynb` | Builds and exports the master session table |
| `H1_analysis.ipynb` | Tests the completion target and compares overall and step-level friction |
| `H2_complete.ipynb` | Compares client success and attempts to first success by age and interface |
| `H4_investor_pschology.ipynb` | Compares journey outcomes across time periods |

### H1 — Session completion and friction

A session qualified as completed if its first recorded step was `start`, its last recorded step was `confirm`, and its duration was **360 seconds or less**. The threshold was based on the **75th percentile of completion times**. We divided qualifying sessions by all recorded sessions in each experiment group.

We used a one-sided test at the 5% significance level and accounted for repeat visits from the same client when estimating uncertainty.

- **H₀** — The Test completion rate is at least 5 percentage points higher than the Control rate.
- **Hₐ** — The Test minus Control difference is below 5 percentage points.

We classified a session as **high-friction** if it met any of these conditions:

- It lasted **more than 300 seconds**, a threshold based on the mean session duration.
- Its first recorded step was something other than `start`.
- It included at least one skip, repeat, or backward move.

Each visit counted once in the overall friction rate, even if it met multiple conditions. A session could qualify as both completed and high-friction. For example, a visit that started at `start`, ended at `confirm`, and lasted 330 seconds met both definitions.

For step-level navigation friction, we calculated the share of visits that reached a step and made at least one irregular move from it. Each visit counted once per step. The `confirm` row captures moves recorded after a confirmation within the same visit.

### H2 — Age and attempts to first success

H2 uses the same session completion rule as H1. We ordered each client’s visits chronologically, counted each visit as one attempt, and identified the first visit that met the completion criteria.

We split clients at the sample median age of **48**. The younger group was under 48, and the older group was 48 or above. The **client success rate** includes all clients with recorded ages and measures the share with at least one successful visit. The **attempts-to-success comparison** includes clients who succeeded at least once.

We used a one-sided Welch’s t-test to assess whether younger successful clients needed fewer attempts on average. We also compared success rates and attempts by age group within each interface and explored account balances and recorded calls descriptively.

### H3 — Market hours, off-hours, and weekends

H3 uses the first recorded event in each visit to group sessions into **weekday market hours** (09:30 to 16:00), **weekday off-hours**, and **weekends**. The analysis assumes the timestamps are in US Eastern Time based on the observed hourly traffic pattern.

We compared completion, friction, and median session duration across the three periods. For the final comparison, H3 will use the same strict completion rule as H1 and H2. The current H3 output counted a confirmation anywhere in the visit toward strict completion. The segmented completion rates will be recalculated using `confirm` as the last recorded step, and the friction and backtracking figures will be checked against the same master session data.

## Key Findings

### H1 — Session completion and friction

| Metric | Control | Test | Test minus Control |
|---|---:|---:|---:|
| Session completion rate | 28.72% | 31.67% | +2.95 percentage points |
| High-friction journey rate | 50.67% | 60.15% | +9.48 percentage points |
| Navigation friction from `step_1` | 12.08% | 22.00% | +9.92 percentage points |
| Navigation friction from `step_3` | 19.94% | 18.33% | −1.61 percentage points |

The one-sided completion test gave **p = 8.527 × 10⁻⁹**. We rejected H₀ at the 5% significance level and concluded that the completion improvement fell short of the **5-percentage-point target**.

The friction comparisons were descriptive. Backward moves had the largest gap among the navigation behaviors (**+6.57 percentage points**). The largest increase in step-level navigation friction occurred at `step_1`. `step_3` was the only step with a lower observed navigation friction rate in Test.

### H2 — Success and attempts by age

| Metric | Under 48 | Age 48 or above |
|---|---:|---:|
| Clients with at least one successful visit | 48.97% | 34.35% |
| Mean attempts among successful clients | 1.149 | 1.250 |
| Median attempts among successful clients | 1 | 1 |

The one-sided Welch’s test supported the hypothesis that younger successful clients needed fewer attempts on average (**p = 1.36 × 10⁻³²**). The difference was about **0.10 attempts**, and the median was one attempt in both groups. The descriptive gap in whether a client succeeded at least once was **14.62 percentage points**.

Both age groups had higher observed client success rates in Test. The age gap was **14.04 percentage points in Control** and **15.04 in Test**. These figures describe the observed gap. The statistical test assessed attempts to success by age.

### H3 — Performance by time period

| Time period | Sessions | Share of sessions |
|---|---:|---:|
| Weekday market hours | 28,530 | 41.08% |
| Weekday off-hours | 26,069 | 37.54% |
| Weekend | 14,848 | 21.38% |

Weekday market hours were the busiest period. The initial H3 run also showed the largest observed completion gain during that period. The final H3 run will verify whether that pattern holds under the shared completion definition.

## Business Recommendations

1. **Refine the redesign before a full rollout.** Session completion rose from **28.72% to 31.67%**, a gain of **2.95 percentage points** against a 5-point target. High-friction journeys also rose from **50.67% to 60.15%**. Run a follow-up A/B test with the same completion target and agree in advance on the highest acceptable friction rate.

2. **Use heatmaps and usability testing to diagnose `step_1`.** Navigation friction at `step_1` rose from **12.08% to 22.00%**, the largest increase of any step. Build a **step-transition heatmap** from the event logs to compare where Test and Control visits moved next, including repeats, backward moves, skips, and exits. In a follow-up study, use **click and scroll heatmaps** to see which controls clients interact with and how far they scroll. Pair these with short usability sessions to decide whether to clarify the main action, revise the instructions, or change when a prompt appears.

3. **Make it easier for clients to reach their first success.** During the experiment, **34.35% of clients aged 48 and above** recorded at least one successful visit, compared with **48.97% of clients under 48**. The median among clients who succeeded was one attempt in both groups. Include clients aged 48 and above in usability testing, and test clearer instructions, a visible progress indicator, or optional help after a repeated step. Separately, check whether visits that begin midway through the process follow an earlier incomplete visit. If that pattern reflects clients returning to finish, test a “Continue where you left off” option.

4. **Measure whether better guidance reduces support costs.** Clients aged 48 and above had slightly more historical calls to Vanguard in both groups. In the next A/B test, capture support contacts related to the online process and track the number of contacts per 100 clients, average handling time, and cost per contact. Use those measures alongside completion rates to see whether clearer instructions and easier access to help improve first-time success and reduce the cost of supporting clients.

5. **Use a phased rollout if the follow-up test meets the success criteria.** Start with a limited release during a lower-traffic period to check stability and confirm that completion and friction are tracking as expected. Then validate the experience during weekday market hours, which accounted for **41.08% of recorded sessions** under the Eastern Time assumption. Review both metrics before expanding to all clients.

## Presentation

The project presentation is available in the `slides/` folder. It summarises the A/B test, highlights the results from H1, H2, and H3 using graphs from the analysis notebooks, and presents the key takeaways and recommendations for Vanguard’s Customer Experience team.

Tableau visualisations of client demographics and other exploratory findings are available in the `figures/` folder.

## Repository Structure

```text
vanguard-project/
├── data/
│   ├── raw/
│   │   ├── df_final_web_data_pt_1.txt      # Original web event log
│   │   ├── df_final_web_data_pt_2.txt      # Original web event log
│   │   ├── df_final_experiment_clients.txt # Original Test & Control assignments
│   │   └── df_final_demo.txt               # Original client demographics
│   └── clean/
│       ├── cleaned_web_data.csv            # Cleaned web events
│       ├── cleaned_client_data.csv         # Cleaned client data
│       ├── sessions_data.csv               # Base session table
│       └── sessions_data_master.csv        # Combined session data
├── notebooks/
│   ├── data_cleanup.ipynb                  # Cleans & combines the source files
│   ├── Cleaned_session_data.ipynb          # Creates the base session table
│   ├── demo_data_analysis.ipynb            # Explores client demographics
│   ├── Session_data_analysis_day_1.ipynb   # Explores timing & session activity
│   ├── journey_status.ipynb                # Checks confirmations and navigation order
│   ├── master_session_data.ipynb           # Builds the master session table
│   ├── H1_analysis.ipynb                   # Analyses H1
│   ├── H2_complete.ipynb                   # Analyses H2
│   └── H3_investor_psychology.ipynb        # Analyses H3
├── figures/                                # Tableau charts
├── slides/                                 # Project presentation
├── config.yaml                             # Project configuration
├── pyproject.toml                          # Python project dependencies
└── README.md                               # Project overview and instructions
```

## How to Run the Project

1. Clone the repository and open the `vanguard-project/` folder in Jupyter Notebook.
2. To reproduce the analysis using the prepared data in `data/clean/`, open the following notebooks in `notebooks/` and run all cells.

   | Notebook | Analysis |
   |---|---|
   | `H1_analysis.ipynb` | Session completion, navigation friction, and the five-percentage-point completion target |
   | `H2_complete.ipynb` | Client success and attempts to first success by age group |
   | `H3_investor_psychology.ipynb` | Session outcomes during market hours, off-hours, and weekends |

3. To rebuild the prepared data from the original Vanguard files in `data/raw/`, run `data_cleanup.ipynb`, `Cleaned_session_data.ipynb`, and `master_session_data.ipynb` in that order before running the analysis notebooks.