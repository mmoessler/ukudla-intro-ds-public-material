# 7.2) Understand data-visualization concepts

---

- Last Update: 2026-09-03
- Source: [07_02_data_visualization_concepts.md](/learning-modules/intro-ds-module/07_02_data_visualization_concepts.md)
- Estimated reading time: 60 minutes
- Estimated activity time: 30 minutes

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Use a question-to-graphic workflow](#use-a-question-to-graphic-workflow)
- [Know the observation grain](#know-the-observation-grain)
- [Match questions and variable types](#match-questions-and-variable-types)
- [Use a grammar of graphics](#use-a-grammar-of-graphics)
- [Choose visual encodings deliberately](#choose-visual-encodings-deliberately)
- [Understand marks and geometric objects](#understand-marks-and-geometric-objects)
- [Treat scales as part of the interpretation](#treat-scales-as-part-of-the-interpretation)
  - [Axis limits](#axis-limits)
  - [Transformations](#transformations)
  - [Comparable scales](#comparable-scales)
- [Make grouping and aggregation explicit](#make-grouping-and-aggregation-explicit)
- [Address overplotting](#address-overplotting)
- [Use facets for repeated comparisons](#use-facets-for-repeated-comparisons)
- [Design colour and text accessibly](#design-colour-and-text-accessibly)
- [Separate exploratory and explanatory graphics](#separate-exploratory-and-explanatory-graphics)
- [Export reproducible figures](#export-reproducible-figures)
- [Interpret without overclaiming](#interpret-without-overclaiming)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
  - [Grammar and implementation](#grammar-and-implementation)
  - [Design and interpretation](#design-and-interpretation)
  - [Accessibility and reproducibility](#accessibility-and-reproducibility)
- [Continue to Application](#continue-to-application)

---

## Learning objectives

After completing this page, you should be able to:

- translate an analytical question into a visual comparison;
- identify the observation grain and variables represented by plotted marks;
- distinguish data, mappings, geometric objects, scales, coordinates, facets, and annotations;
- select appropriate encodings for quantitative and categorical variables;
- explain how axes, transformations, aggregation, and overplotting affect interpretation;
- design graphics that remain readable and accessible;
- distinguish exploratory iteration from explanatory refinement;
- save a figure reproducibly with documented dimensions and output paths; and
- write a qualified interpretation that distinguishes pattern, association, and causation.

---

## Place in the session

This is the **Concepts** part of the Data Visualization session:

~~~text
Motivation  →  Concepts  →  Application
                ↑
             this page
~~~

[Why visualize data?](07_01_data_visualization_motivation.md) explains why visual choices
matter. This page provides a general decision model, followed by a
[maize-yield worked example](07_03_data_visualization_application.md).

Use one central question throughout:

> Which comparison should the viewer make, and which visual choices make that
> comparison accurate, visible, and interpretable?

---

## Use a question-to-graphic workflow

Avoid starting with, “Which chart should I make?” Begin with:

1. **Purpose:** Are you investigating or communicating?
2. **Question:** Is the focus a distribution, comparison, change, or relationship?
3. **Data:** Which artifact, population, grain, variables, and units are relevant?
4. **Encoding:** Which positions and other visual properties represent variables?
5. **Review:** Could scales, grouping, missingness, or design create a false impression?
6. **Interpretation:** What does the graphic support, and what remains unresolved?

This sequence is not strictly linear: a plot can reveal a new question, a
metadata issue, or a need for a different representation. Record consequential
changes in code rather than only through an interactive interface.

---

## Know the observation grain

The **grain** states what one row represents. A plotted mark may represent one
row, several rows summarized together, or part of a row after reshaping.

For the integrated maize data, one row represents one project country-year,
including a growing season ending in that year — so in a scatterplot with
yield and precipitation on the axes, one point represents one country-year.

In a plot of average yield by country, one point represents 33 country-year
rows summarized by a mean: a different analytical object. Always be able to
complete this sentence:

> One mark in this graphic represents ...

If the sentence is unclear, return to the data preparation contract and the
plotting code.

---

## Match questions and variable types

Different structures suggest different starting graphics:

| Question | Variables | Useful starting graphics |
| --- | --- | --- |
| How are values distributed? | One quantitative variable | Histogram, density plot, empirical distribution, boxplot |
| How do groups compare? | Quantitative value and categorical group | Aligned points, boxplots, violins, facets |
| How does a measure change? | Quantitative value and ordered time | Points and lines |
| How do two measures vary together? | Two quantitative variables | Scatterplot |
| How many observations belong to categories? | One or more categorical variables | Bars or dot plots |
| Where does a measure occur? | Measure and valid spatial geometry | Map, often alongside non-spatial comparisons |

These are starting points, not rules that guarantee validity: a boxplot can
hide multimodality, a line can imply continuity between sparse observations,
and a map makes area visually prominent even when area is not the quantity of
interest.

---

## Use a grammar of graphics

A grammar of graphics treats a plot as explicit components:

| Component | Question | `ggplot2` example |
| --- | --- | --- |
| Data | Which observations enter the graphic? | `ggplot(data = maize)` |
| Aesthetic mapping | Which variables control visual properties? | `aes(x = year, y = yield)` |
| Geometry | Which marks represent observations or summaries? | `geom_point()`, `geom_line()` |
| Statistical transformation | Is a count, bin, smooth, or summary calculated? | `stat_bin()`, `stat_summary()` |
| Scale | How do data values map to positions or colours? | `scale_y_log10()` |
| Coordinate system | How are scales arranged in the display? | `coord_cartesian()` |
| Facet | Which groups receive repeated panels? | `facet_wrap(vars(country))` |
| Labels and theme | How is meaning communicated? | `labs()`, `theme_minimal()` |

In `ggplot2`, a property inside `aes()` is mapped to a variable. A property
outside `aes()` is fixed:

~~~r
# Map a scientifically relevant group to colour
geom_line(aes(colour = country))

# Use one fixed colour when grouping is not encoded
geom_line(colour = "#29508A")
~~~

This distinction prevents accidental legends and makes the visual meaning
inspectable in code.

---

## Choose visual encodings deliberately

People compare some visual properties more accurately than others. For precise
quantitative comparison, prefer:

1. position on a common scale;
2. position on aligned scales;
3. length; then
4. angle, area, volume, or colour intensity.

Use **colour hue** to distinguish a manageable number of categories, and an
ordered lightness scale for ordered or continuous values when the palette
supports it. Avoid an arbitrary rainbow scale: perceived colour differences do
not follow equal numeric differences, so transitions can appear where none
exist.

Use **size** carefully. Viewers perceive the area of a mark, so doubling its
radius more than doubles its visible area. Let the plotting scale calculate
area consistently.

Use **shape** for few categories with enough mark size to recognize it, and
encode important distinctions redundantly — through position, label, shape, or
line type — rather than colour alone.

---

## Understand marks and geometric objects

Marks carry different implications:

- **points** show individual observations;
- **lines** connect ordered observations and emphasize continuity or change;
- **bars** encode magnitude through length and generally require a meaningful zero baseline;
- **areas** emphasize totals or composition but can make internal comparisons difficult;
- **boxes and violins** summarize distributions and should be supported by sufficient observations;
- **smooths** summarize an estimated pattern and introduce method choices.

A line joining countries is inappropriate because country order is not
continuous; a line joining annual observations within one country is
meaningful once observations are grouped by country and ordered by year.

A histogram is not a picture of the raw distribution alone. It applies a
statistical transformation that assigns values to bins and counts them. Inspect
whether conclusions change across reasonable bin widths.

---

## Treat scales as part of the interpretation

Scales determine how numeric and categorical values become visible distances,
colours, sizes, or labels.

### Axis limits

Restricting a scale can exaggerate differences or remove data before a
statistical calculation. `coord_cartesian()` can zoom without dropping
observations, but the narrower view still needs justification.

For bars, a non-zero baseline breaks the relation between length and magnitude.
For points and lines, zero is not always required, but the selected range must
support an honest interpretation.

### Transformations

A logarithmic axis can make multiplicative change and wide ranges easier to
inspect. Equal visual distances then represent equal ratios rather than equal
absolute differences. Name the transformation and keep physical units
available for interpretation.

### Comparable scales

Fixed scales across facets support magnitude comparison; free scales can
reveal within-panel patterns when groups have very different ranges but
prevent direct comparison of absolute levels. State which comparison matters.

---

## Make grouping and aggregation explicit

Grouping affects both calculations and visual connections. In a country time
series, lines must be grouped by country. In a summary, `group_by(country)`
changes one dataset-wide statistic into one statistic per country.

Aggregation requires a contract:

- Which rows enter each group?
- Which statistic is calculated?
- Are observations weighted?
- How are missing values handled?
- What variation disappears?
- What does each output mark represent?

Do not calculate a mean with `na.rm = TRUE` without reporting how many values
were missing — a national mean and a mean across countries can imply
different weights, and visualization does not remove these statistical
choices.

---

## Address overplotting

**Overplotting** occurs when marks overlap and hide observation frequency.
Possible responses include:

- smaller marks;
- transparency with `alpha`;
- jitter for discrete or rounded positions;
- two-dimensional bins or contours;
- faceting by a meaningful group; or
- summarizing after the aggregation rule is stated.

Each response has limitations: transparency depends on display background and
export format, jitter adds visual displacement that is not measurement error,
bins trade precise locations for counts, and faceting can make cross-panel
comparison harder. Choose the response that supports the question.

---

## Use facets for repeated comparisons

**Small multiples** repeat the same visual structure for groups. Faceting the
maize time series by country avoids nine overlapping lines and lets readers
inspect patterns consistently.

Review:

- whether all panels use the same variables and transformations;
- whether fixed or free scales match the comparison;
- whether panel order supports interpretation;
- whether empty panels or missing periods are meaningful; and
- whether labels fit at the intended export size.

Facets are often preferable to mapping many categories to similar colours.
They use position and repetition, but require enough space.

---

## Design colour and text accessibly

Accessibility is part of analytical communication, not final decoration.

- Use palettes with sufficient contrast and colour-vision-deficiency support.
- Do not rely on red versus green alone.
- Use readable text at the final display size.
- Prefer direct labels when they reduce legend lookup.
- Provide units on quantitative axes.
- Use concise titles that state the subject, not an unsupported conclusion.
- Add captions or nearby prose describing source, grain, and limitations.
- Provide alternative text or an equivalent textual interpretation in the publication context.

Test a saved figure, not only the large interactive preview. A graphic that is
readable on a laptop may fail when placed in a report or presentation.

---

## Separate exploratory and explanatory graphics

Exploratory graphics can be numerous and provisional. They help inspect:

- distributions and missingness;
- alternative scales and groupings;
- anomalies and possible data problems;
- sensitivity to bins or aggregation; and
- questions that need descriptive or modeled analysis.

An explanatory graphic should focus on one comparison. Refine it by removing
irrelevant elements, emphasizing the intended evidence, adding context, and
writing a qualified caption.

Do not choose the most dramatic exploratory view after trying many
alternatives without acknowledging the selection: a communication graphic
should follow a defensible question, not maximize apparent difference.

---

## Export reproducible figures

A reproducible figure has a traceable input, script, environment, and output
path. Save it through code:

~~~r
ggsave(
  filename = here("figures", "maize-yield-trends.png"),
  plot = yield_plot,
  width = 10,
  height = 7,
  units = "in",
  dpi = 300
)
~~~

Choose format according to use:

- PNG is suitable for raster display and web use;
- PDF or SVG preserves vector marks and text when supported; and
- dimensions determine layout and readability independently of resolution.

Use stable, descriptive filenames. Do not overwrite a different conceptual
figure merely because it is the latest plot. Follow the repository policy for
tracked and generated figures.

---

## Interpret without overclaiming

Separate three levels:

1. **Visible pattern:** “Yield values are higher in later years for several countries.”
2. **Descriptive association:** “Higher precipitation observations coincide with different yield values in this sample.”
3. **Causal claim:** “Increasing precipitation causes yield to rise.”

The first describes the graphic. The second requires a defined descriptive
analysis. The third requires a causal design and assumptions not supplied by a
scatterplot.

For the maize project, country effects, long-term trends, measurement error,
spatial aggregation, irrigation, inputs, and temperature can affect a visible
rainfall-yield pattern; keep these limitations attached to the visual story.

---

## Check your understanding

1. What should you state before selecting a chart form?
2. What can one point represent in the integrated maize scatterplot?
3. How does a mapped aesthetic differ from a fixed property in `ggplot2`?
4. Why is position generally preferred for precise quantitative comparison?
5. When is a line an inappropriate mark?
6. How can axis limits and log transformations alter interpretation?
7. What information must accompany an aggregated mark?
8. Which approaches can address overplotting, and what does each sacrifice?
9. When are free facet scales useful, and what comparison do they prevent?
10. What makes a saved figure reproducible and accessible?
11. How does a visible association differ from a causal claim?

---

## Further resources

### Grammar and implementation

- [R for Data Science (2e): Data visualization](https://r4ds.hadley.nz/data-visualize.html)
- [ggplot2 documentation](https://ggplot2.tidyverse.org/)
- Hadley Wickham, Danielle Navarro, and Thomas Lin Pedersen,
  [ggplot2: Elegant Graphics for Data Analysis](https://ggplot2-book.org/)

### Design and interpretation

- Claus O. Wilke, [Fundamentals of Data Visualization](https://clauswilke.com/dataviz/)
- Kieran Healy, [Data Visualization: A Practical Introduction](https://socviz.co/)
- [Data-to-Viz](https://www.data-to-viz.com/)

### Accessibility and reproducibility

- [The Turing Way: Data visualisation](https://book.the-turing-way.org/communication/visualisation/)
- [Viridis colour scales for `ggplot2`](https://ggplot2.tidyverse.org/reference/scale_viridis.html)
- [W3C: Images tutorial](https://www.w3.org/WAI/tutorials/images/)

---

## Continue to Application

Continue with [Visualize maize yield and
precipitation](07_03_data_visualization_application.md). You will formulate plot
contracts, inspect the prepared artifacts, generate complementary exploratory
views, refine one communication graphic, export it through code, and review
the claims it supports.
