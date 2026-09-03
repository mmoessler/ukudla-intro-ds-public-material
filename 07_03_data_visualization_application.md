# 7.3) Apply data visualization: maize-yield worked example

---

- Last Update: 2026-09-03
- Source: [07_03_data_visualization_application.md](/learning-modules/intro-ds-module/07_03_data_visualization_application.md)
- Estimated completion time: 6–8 hours
- Independent extension: 2–3 hours
- Prerequisites: Motivation and Concepts pages; completed Data Preparation workflow
- Required output: plotting script, exploratory figure set, communication figure, and visual interpretation

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Scenario and deliverables](#scenario-and-deliverables)
  - [Transfer the workflow to another study design](#transfer-the-workflow-to-another-study-design)
- [Before you begin](#before-you-begin)
- [1. State visual questions and plot contracts](#1-state-visual-questions-and-plot-contracts)
- [2. Inspect the analytical artifacts](#2-inspect-the-analytical-artifacts)
- [3. Examine the yield distribution](#3-examine-the-yield-distribution)
- [4. Compare yield trends across countries](#4-compare-yield-trends-across-countries)
- [5. Examine growing-season precipitation](#5-examine-growing-season-precipitation)
- [6. Explore yield and precipitation together](#6-explore-yield-and-precipitation-together)
- [7. Refine one communication graphic](#7-refine-one-communication-graphic)
- [8. Export and review the figure artifacts](#8-export-and-review-the-figure-artifacts)
- [Independent extension](#independent-extension)
- [Troubleshooting](#troubleshooting)
- [Completion checklist](#completion-checklist)
- [Reflect on the application](#reflect-on-the-application)
- [Further resources](#further-resources)

---

## Learning objectives

After completing this exercise, you should be able to:

- formulate visual questions and state what one plotted mark represents;
- inspect plotting inputs, grain, variables, units, and missingness;
- create distribution, time-series, and relationship graphics with `ggplot2`;
- compare alternative bins, groupings, scales, and facets;
- identify and respond to overplotting;
- refine an exploratory chart into an accessible communication graphic;
- export figures reproducibly; and
- write a visual interpretation separating pattern from causal claims.

---

## Place in the session

This is the **Application** part of the Data Visualization session:

~~~text
Motivation  →  Concepts  →  Application
                              ↑
                           this page
~~~

Before beginning, review [Why visualize data?](07_01_data_visualization_motivation.md)
and [Understand data-visualization concepts](07_02_data_visualization_concepts.md).

The preceding topic produced a documented maize country-year panel and
preparation audit; this exercise treats those as plotting inputs and does not
acquire, repair, or edit data.

---

## Scenario and deliverables

> **Worked-example scope:** The supplied figures use countries, years, yield,
> and precipitation. Treat these as one implementation of a general visual
> workflow: define the comparison, identify the observation grain, choose an
> encoding, inspect alternatives, and communicate limitations.

The project team wants a small visual story that answers:

> How do maize yield and growing-season precipitation vary across the selected
> countries and years, and which patterns should later be quantified?

The required deliverables are:

~~~text
scripts/visualize-maize-data.R
figures/maize-yield-distribution.png
figures/maize-yield-trends.png
figures/growing-season-precipitation-trends.png
figures/yield-versus-precipitation.png
figures/growing-season-precipitation-distribution-by-country.png
~~~

### Transfer the workflow to another study design

| Project context | Useful first views | Structure to keep visible |
| --- | --- | --- |
| Laboratory | Distributions, replicate agreement, calibration or control plots | Batch, assay, sample, replicate type, detection limits |
| Field experiment | Treatment distributions, block/site panels, outcome by treatment | Assignment, blocks, replicates, repeated occasions |
| Field observation | Coverage maps, group distributions, trajectories, exposure-outcome plots | Sampling groups, sites, repeated units, missing coverage |
| Secondary data | Entity comparisons, time series, maps, variable relationships | Reporting grain, revisions, flags, aggregation and spatial support |

Replace country facets with scientifically relevant groups such as treatment,
batch, block, site, sampling stratum, or provider entity. A grouping variable
should reflect the study question and design rather than merely produce a
visually convenient layout.

The first four figures support exploration; the final figure should focus on
one comparison for communication. Also provide a short interpretation that
states:

- the visible pattern;
- population and observation grain;
- relevant units and aggregation;
- at least two limitations; and
- one question for the next session.

Do not select a dramatic title before inspecting the evidence. Do not claim
that precipitation causes yield differences.

---

## Before you begin

Work from the standalone `maize-yield-project` repository. Confirm the branch
and working-tree state:

~~~bash
pwd
git status --short --branch
~~~

Restore the recorded environment and recreate the required data:

~~~bash
Rscript scripts/setup.R
Rscript scripts/validate-data.R
Rscript scripts/prepare-maize-data.R
Rscript scripts/integrate-data.R
~~~

Inspect the documentation before plotting:

| Role | File |
| --- | --- |
| Prepared maize panel documentation | `docs/data/maize-yield-panel.md` |
| Prepared panel dictionary | `metadata/maize-yield-panel-data-dictionary.csv` |
| Integrated-data documentation | `docs/data/maize-yield-with-precipitation.md` |
| Integrated-data dictionary | `metadata/maize-yield-with-precipitation-data-dictionary.csv` |
| Preparation audit | `results/tables/data-preparation-audit.csv` |
| Integration audit | `results/tables/data-integration-audit.csv` |

Both audits must pass before their outputs are treated as ready; the workflow
uses fixed snapshots and requires no network access.

Create `scripts/visualize-maize-data.R` with the common setup:

~~~r
# Create reproducible visualizations of maize yield and precipitation.

source("scripts/functions.R")

assert_project_root()
ensure_project_directories()
check_required_packages(c("dplyr", "ggplot2", "here", "readr"))

library(dplyr)
library(ggplot2)
library(here)
library(readr)

panel_file <- here("data", "derived", "maize-yield-panel.csv")
integrated_file <- here(
  "data", "derived", "maize-yield-with-precipitation.csv"
)

required_files <- c(panel_file, integrated_file)
missing_files <- required_files[!file.exists(required_files)]
if (length(missing_files) > 0) {
  stop("Required derived data not found: ", paste(missing_files, collapse = ", "), call. = FALSE)
}

maize <- read_csv(panel_file, show_col_types = FALSE)
integrated <- read_csv(integrated_file, show_col_types = FALSE)

save_figure <- function(name, plot, width = 10, height = 7) {
  ggsave(here("figures", name), plot, width = width, height = height, units = "in", dpi = 300)
}
~~~

`save_figure()` centralizes the export settings shared by every figure below.

---

## 1. State visual questions and plot contracts

Before coding, complete this table:

| Figure | Question | One mark represents | Variables | Scale or grouping decision |
| --- | --- | --- | --- | --- |
| Yield distribution | How are annual yield values distributed? | One bin count | Yield | Bin width |
| Yield trends | How does yield change within countries? | One country-year joined in time | Year, yield, country | Fixed or free facets |
| Precipitation | How does seasonal precipitation vary? | Define this | Define this | Define this |
| Relationship | How do yield and precipitation vary together? | One country-year | Yield, precipitation, country | Transparency and facets |

For every figure, state:

- purpose, audience, and input artifact;
- population, observation grain, and plotted variables and units;
- filtering and missing-value rule;
- statistical transformation, grouping, and scale choices; and
- claim boundary.

The plot contract can be a Markdown table in a project note; it need not
become another machine-readable metadata system.

---

## 2. Inspect the analytical artifacts

Do not infer meaning only from column names:

~~~r
glimpse(maize)
glimpse(integrated)

nrow(maize)
nrow(integrated)

maize |>
  summarise(
    countries = n_distinct(country),
    first_year = min(year),
    last_year = max(year),
    missing_yield = sum(is.na(yield_tonnes_per_hectare))
  )

integrated |>
  summarise(
    countries = n_distinct(project_country_id),
    first_year = min(year),
    last_year = max(year),
    missing_precipitation =
      sum(is.na(growing_season_precipitation_mm))
  )
~~~

Confirm 297 unique country-year rows in each derived artifact, and review the
dictionaries for `yield_tonnes_per_hectare` and
`growing_season_precipitation_mm`. The precipitation value is a seasonal total
of country-area mean daily CHIRPS estimates, not rainfall measured at maize
fields.

---

## 3. Examine the yield distribution

Begin with a histogram:

~~~r
yield_distribution <- ggplot(
  maize,
  aes(x = yield_tonnes_per_hectare)
) +
  geom_histogram(
    binwidth = 0.25,
    boundary = 0,
    colour = "white",
    fill = "#29508A",
    na.rm = TRUE
  ) +
  labs(
    title = "Distribution of annual maize yield",
    subtitle = "Nine selected countries, 1990–2022",
    x = "Yield (tonnes per hectare)",
    y = "Country-year observations",
    caption = paste(
      "Source: fixed FAOSTAT teaching sample.",
      "One observation represents one country-year."
    )
  ) +
  theme_minimal(base_size = 11)

yield_distribution
~~~

Compare at least three reasonable bin widths and record what remains stable: a
histogram displays counts after binning and does not preserve the precise
position of every value.

Then ask whether pooling countries answers the intended question, create a
faceted version, and avoid interpreting a pooled shape as the distribution of
a single homogeneous population.

Save the selected exploratory version:

~~~r
save_figure("maize-yield-distribution.png", yield_distribution, width = 9, height = 6)
~~~

---

## 4. Compare yield trends across countries

Use small multiples rather than nine overlapping coloured lines:

~~~r
yield_trends <- ggplot(
  maize,
  aes(x = year, y = yield_tonnes_per_hectare)
) +
  geom_line(colour = "#29508A", linewidth = 0.5, na.rm = TRUE) +
  geom_point(colour = "#29508A", size = 0.8, na.rm = TRUE) +
  facet_wrap(vars(country), ncol = 3) +
  labs(
    title = "Maize yield over time",
    subtitle = "Panels share a common yield scale",
    x = "Year",
    y = "Yield (tonnes per hectare)",
    caption = "Source: fixed FAOSTAT teaching sample."
  ) +
  theme_minimal(base_size = 10) +
  theme(panel.grid.minor = element_blank())

yield_trends
~~~

Explain why grouping works even though `group = country` is not explicit: each
facet contains one country. Create a second version using
`scales = "free_y"`. Compare the two:

- Which makes absolute country levels easier to compare?
- Which makes within-country change easier to see?
- Could a reader overlook the scale difference?

Retain fixed scales for the required comparison unless you justify another
choice clearly.

~~~r
save_figure("maize-yield-trends.png", yield_trends)
~~~

---

## 5. Examine growing-season precipitation

Design a figure that answers one precipitation question. For example:

~~~r
precipitation_plot <- ggplot(
  integrated,
  aes(x = growing_season_precipitation_mm)
) +
  geom_histogram(
    binwidth = 100,
    boundary = 0,
    colour = "white",
    fill = "#3B8B47",
    na.rm = TRUE
  ) +
  facet_wrap(vars(project_country_name), ncol = 3) +
  labs(
    title = "Growing-season precipitation distributions",
    subtitle = "October–April seasons ending in 1990–2022",
    x = "Country-area seasonal precipitation (mm)",
    y = "Seasons",
    caption = paste(
      "Source: CHIRPS v2 via ClimateSERV.",
      "Country-area estimates are not maize-field exposure."
    )
  ) +
  theme_minimal(base_size = 10)

precipitation_plot
~~~

Inspect sensitivity to bin width and shared versus free scales, and explain
why the value should not be described simply as “rainfall received by maize.”

~~~r
save_figure(
  "growing-season-precipitation-distribution-by-country.png",
  precipitation_plot
)
~~~

Complement the distribution with a time-ordered view so shifts, cycles, gaps,
and unusual seasons remain visible:

~~~r
precipitation_trends <- ggplot(
  integrated,
  aes(x = year, y = growing_season_precipitation_mm)
) +
  geom_line(colour = "#3B8B47", linewidth = 0.5, na.rm = TRUE) +
  geom_point(colour = "#3B8B47", size = 0.8, na.rm = TRUE) +
  facet_wrap(vars(project_country_name), ncol = 3) +
  labs(
    title = "Growing-season precipitation over time",
    x = "Season-ending year",
    y = "Country-area seasonal precipitation (mm)"
  ) +
  theme_minimal(base_size = 10)

save_figure("growing-season-precipitation-trends.png", precipitation_trends)
~~~

---

## 6. Explore yield and precipitation together

Create a scatterplot in which one point represents one country-year:

~~~r
yield_precipitation <- ggplot(
  integrated,
  aes(
    x = growing_season_precipitation_mm,
    y = yield_tonnes_per_hectare
  )
) +
  geom_point(
    colour = "#29508A",
    alpha = 0.55,
    size = 1.3,
    na.rm = TRUE
  ) +
  facet_wrap(vars(project_country_name), ncol = 3) +
  labs(
    title = "Maize yield and growing-season precipitation",
    subtitle = "Each point represents one country-year",
    x = "Country-area seasonal precipitation (mm)",
    y = "Maize yield (tonnes per hectare)",
    caption = paste(
      "Sources: FAOSTAT and CHIRPS v2 via ClimateSERV.",
      "The graphic describes association, not causation."
    )
  ) +
  theme_minimal(base_size = 10) +
  theme(panel.grid.minor = element_blank())

yield_precipitation
~~~

Compare this to a pooled plot coloured by country. Discuss:

- whether points overlap;
- whether the pooled view confounds country differences with within-country variation;
- whether a few high-yield countries dominate attention;
- what transparency makes visible; and
- which patterns require numerical description or modeling.

Do not add a fitted line merely as decoration: a smooth or regression line
introduces method, functional-form, grouping, and uncertainty choices that
become central in later sessions.

~~~r
save_figure("yield-versus-precipitation.png", yield_precipitation)
~~~

---

## 7. Refine one communication graphic

Choose one exploratory result and state one intended comparison. The
communication figure may build on the yield trends but must be reviewed as a
separate artifact.

Use this refinement sequence:

1. Write a one-sentence message supported by visible evidence.
2. Identify the audience and final display size.
3. Remove encodings and decoration unrelated to the comparison.
4. Use fixed scales when comparing magnitudes across panels.
5. Add title, subtitle, units, source, grain, and limitation.
6. Check contrast, text size, panel labels, and grayscale readability.
7. Ask another learner to state the message without your explanation.
8. Revise the graphic if their interpretation differs from the intended one.

Do not use a title such as “Rainfall increases maize yield”; a defensible
title describes the variables and scope, while the accompanying prose states
a visible pattern and its uncertainty. Revise the title if your evidence
supports a more precise, qualified statement.

Prototype the refinement in code and compare it with the exploratory version:

~~~r
communication_plot <- yield_trends +
  labs(
    title = "Maize-yield trajectories differ across countries",
    subtitle = "Annual country-level observations, 1990–2022",
    caption = paste(
      "Source: fixed FAOSTAT teaching sample.",
      "National observations do not show within-country variation."
    )
  )

~~~

The project need not retain a duplicate figure when the refined version does
not serve a distinct purpose. If it does, add a clearly named output to the
script and artifact registry rather than exporting it manually.

---

## 8. Export and review the figure artifacts

Run the complete script from the project root:

~~~bash
Rscript scripts/visualize-maize-data.R
~~~

Confirm that all expected files exist and are non-empty:

~~~bash
ls -lh figures/*.png
~~~

Open each saved figure at its intended size. Review:

- title, subtitle, labels, units, caption, and source;
- clipping, overlap, contrast, text size, and facet scales;
- represented observations and missing values;
- consistency with the plot contract; and
- whether rerunning the script recreates the same outputs.

Inspect version-control status:

~~~bash
git status --short
git diff -- scripts/visualize-maize-data.R
~~~

Follow the project figure policy: exploratory artifacts need not all be
retained, but their code and the selected communication figure should be
reviewable, and temporary or manually exported duplicates should not be
committed.

Write a short interpretation with this structure:

~~~text
Figure purpose:
Visible pattern:
Population and grain:
Scale or aggregation choices:
What the figure does not establish:
Question for descriptive analysis:
~~~

---

## Independent extension

Choose one extension and justify every new visual choice:

1. Compare a physical and log-yield scale, and explain how equal distances
   change meaning.
2. Build an accessible two-country comparison using both colour and line type.
3. Compare country-level rainfall distributions using aligned dot plots
   instead of histograms.
4. Investigate one unusual observation using the FAOSTAT flag and source
   documentation, without deleting it.
5. Export a vector PDF of the communication figure and compare its
   suitability with PNG.

Add a short note on what the extension reveals, which choices it depends on,
and why it does not change the claim boundary.

---

## Troubleshooting

| Problem | Fix |
| --- | --- |
| Derived input missing | Run the validated preparation and integration scripts in order; do not point the plotting script at an undocumented substitute. |
| Line connects different countries | Check grouping — use `group = country`, map country to an aesthetic, or facet so each panel holds one country. |
| Histogram changes with bin width | Expected near the bin scale; compare reasonable widths rather than treating one as definitive. |
| Points appear missing | Check missing values, axis limits, transformations, and overplotting; count rows supplied to the geometry. |
| Facets look similar despite different values | Check whether `scales = "free_y"` is active — free scales support shape comparison, not magnitude comparison. |
| Labels or panels clipped in the PNG | Increase dimensions, shorten labels, or revise the layout; higher DPI alone adds no layout space. |
| Script changes the input data | Stop — visualization must read derived data and write figures, not overwrite managed or derived inputs. |

---

## Completion checklist

- [ ] Every figure begins with a stated question and audience.
- [ ] Input artifact, population, grain, variables, and units are documented.
- [ ] One plotted mark or aggregate can be described precisely.
- [ ] Distribution results were checked across reasonable bin choices.
- [ ] Trend grouping connects observations only within countries.
- [ ] Fixed and free facet scales were compared deliberately.
- [ ] Overplotting and missingness were inspected.
- [ ] Titles, axes, units, captions, and sources are readable.
- [ ] Important distinctions do not depend on colour alone.
- [ ] The script recreates figures through project-relative paths.
- [ ] The communication graphic supports one qualified message.
- [ ] Interpretation distinguishes pattern, association, and causation.
- [ ] One question is carried forward to Descriptive Data Analysis.

---

## Reflect on the application

1. Which visual question was easiest to formulate, and which remained ambiguous?
2. What does one mark represent in each of your figures, and what was lost when observations were aggregated?
3. Which histogram features remained stable across bin widths, and how did fixed versus free facet scales change country comparisons?
4. What did transparency or faceting reveal about overlapping observations?
5. How did you make the communication figure accessible, and which title did you reject because it overstated the evidence?
6. What can the yield-precipitation scatterplot show and not show, and which visible pattern should be quantified next?

---

## Further resources

- [R for Data Science (2e): Data visualization](https://r4ds.hadley.nz/data-visualize.html)
- [ggplot2 reference](https://ggplot2.tidyverse.org/reference/)
- Claus O. Wilke, [Fundamentals of Data Visualization](https://clauswilke.com/dataviz/)
- Kieran Healy, [Data Visualization: A Practical Introduction](https://socviz.co/)
- [The Turing Way: Data visualisation](https://book.the-turing-way.org/communication/visualisation/)
- [Viridis colour scales](https://ggplot2.tidyverse.org/reference/scale_viridis.html)
- [W3C: Images tutorial](https://www.w3.org/WAI/tutorials/images/)
