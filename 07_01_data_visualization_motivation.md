# 7.1) Why visualize data?

---

- Last Update: 2026-09-03
- Source: [07_01_data_visualization_motivation.md](/learning-modules/intro-ds-module/07_01_data_visualization_motivation.md)
- Estimated reading time: 20 minutes
- Estimated activity time: 10 minutes

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Prepared data do not reveal patterns by themselves](#prepared-data-do-not-reveal-patterns-by-themselves)
- [A graphic is an analytical choice](#a-graphic-is-an-analytical-choice)
- [Exploration and communication have different purposes](#exploration-and-communication-have-different-purposes)
- [Visualization supports quality review](#visualization-supports-quality-review)
- [What can go wrong](#what-can-go-wrong)
  - [The chart form does not match the question](#the-chart-form-does-not-match-the-question)
  - [A truncated scale exaggerates a small difference](#a-truncated-scale-exaggerates-a-small-difference)
  - [Aggregation hides the relevant variation](#aggregation-hides-the-relevant-variation)
  - [Colour carries meaning that some readers cannot access](#colour-carries-meaning-that-some-readers-cannot-access)
  - [Overplotting creates false absence](#overplotting-creates-false-absence)
  - [A visual association is described as causal](#a-visual-association-is-described-as-causal)
- [Across study designs and in the worked example](#across-study-designs-and-in-the-worked-example)
- [Initial activity](#initial-activity)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- explain why visual representations complement tables and numerical summaries;
- describe a chart as a sequence of analytical and communication choices;
- distinguish exploratory from explanatory visualization;
- identify ways in which scales, aggregation, filtering, and design can mislead;
- explain how visualization supports data-quality investigation without proving an error; and
- formulate visual questions for the maize-yield and precipitation data.

---

## Place in the session

This is the **Motivation** part of the Data Visualization session:

~~~text
Motivation  →  Concepts  →  Application
    ↑
 this page
~~~

Data Preparation created documented analytical observations. This session asks
how to map observations to visual properties so patterns can be inspected and
communicated without hiding their meaning, study design, or limitations.

[Understand data-visualization concepts](07_02_data_visualization_concepts.md) develops
the question-to-graphic workflow. The [application](07_03_data_visualization_application.md)
demonstrates it in the maize project and provides transfer guidance.

---

## Prepared data do not reveal patterns by themselves

A prepared table preserves exact values, documented variables, units,
observation grain, and candidate keys, but a reader cannot easily recognize
297 country-year observations by scanning a CSV file.

Visual representations can make several structures easier to inspect:

- the shape and range of a distribution;
- differences between countries;
- changes and interruptions over time;
- unusual observations;
- missing periods; and
- possible relationships between yield and precipitation.

Visualization does not replace the table: a chart selects and encodes parts of
it for a particular question, and useful visualization deliberately reduces
complexity rather than displaying every variable and limitation.

---

## A graphic is an analytical choice

A chart is not a neutral photograph of a dataset. Before any mark appears, an
analyst has already decided:

- which observations and variables to include;
- whether marks show observations or summaries;
- which variables control position, colour, size, shape, or facets;
- where axes begin and end and whether scales are transformed; and
- which labels, annotations, dimensions, and resolution the audience sees.

These choices determine which comparisons are easy and which variation becomes
invisible. For example, showing one mean yield per country answers a different
question from showing all annual yields, and free facet scales that ease
within-country reading also prevent direct magnitude comparison.

The relevant question is therefore not only, “Is the code correct?” It is also,
“Does this visual representation support the intended comparison honestly?”

---

## Exploration and communication have different purposes

**Exploratory visualization** supports investigation, usually for the analyst
or project team. Many temporary graphics may be created to inspect
distributions, missingness, group differences, or alternative scales,
generating questions and testing interpretations.

**Explanatory visualization** supports communication: it selects a defensible
finding for a defined audience and generally needs a clear title, readable
labels, visible units, source information, and a statement of limitations.

The two purposes form a workflow:

~~~text
question
   ↓
several exploratory views
   ↓
inspection and revision
   ↓
one focused communication graphic
   ↓
caption and qualified interpretation
~~~

An explanatory figure should remain traceable to prepared data and
reproducible code. Manual adjustments that cannot be recreated weaken that
traceability.

---

## Visualization supports quality review

Plots can reveal observations that deserve investigation. A time series may
show a jump, a histogram an unexpected boundary, or a scatterplot repeated
values.

These patterns are signals, not verdicts. An unusual value may represent:

- a source or parsing error;
- a change in definition or reporting practice;
- a real drought, policy change, or agricultural event;
- an artifact of aggregation; or
- legitimate variation.

Return to metadata, flags, provenance, and preparation decisions before
correcting anything: a plot can motivate a validation question but cannot
prove that an observation is wrong.

---

## What can go wrong

### The chart form does not match the question

A chart form should not be chosen merely because it is familiar; begin with
the comparison and variable types.

### A truncated scale exaggerates a small difference

Bars encode magnitude through length and rely on a meaningful zero baseline.
If an axis is restricted, make the choice visible and justify it.

### Aggregation hides the relevant variation

A country average can conceal changes across years, and a national value can
conceal within-country differences. State what one mark represents and how it
was calculated.

### Colour carries meaning that some readers cannot access

Avoid palettes with poor contrast or categories distinguished only by red and
green; combine colour with position, labels, line type, or shape where
appropriate.

### Overplotting creates false absence

Many observations at the same location can appear as one point. Transparency,
smaller marks, jitter, bins, or facets can reveal density, but each changes
the view.

### A visual association is described as causal

Yield and precipitation may vary together because of rainfall, country
differences, trends, omitted variables, or coincidence. A scatterplot
establishes neither causal direction nor a controlled comparison.

---

## Across study designs and in the worked example

Relevant comparisons might be between laboratory controls and samples,
treatments and blocks in an experiment, sites or visits in an observational
study, or entities and periods in secondary data. The visual grammar is shared;
the grouping, replication, uncertainty, and dependence that must remain visible
come from the study design.

The maize-yield project provides one longitudinal secondary-data example.

The example project provides two useful analytical artifacts:

- `data/derived/maize-yield-panel.csv`, with country-year maize measures; and
- `data/derived/maize-yield-with-precipitation.csv`, which adds CHIRPS
  October-April country-area precipitation.

The existing exploration script creates a faceted maize-yield time series;
this session expands it into a coherent visual investigation:

1. inspect the maize-yield distribution and cross-country trends;
2. inspect growing-season precipitation and its relationship to yield;
3. compare alternative encodings and scales; and
4. refine one chart for communication.

Every output must retain the documented units and country-year grain. The
figures can motivate later descriptive or modeled analysis, but must not
claim that precipitation alone explains changes in maize yield.

---

## Initial activity

Before reading the Concepts page, write one visual question for each structure:

| Structure | Your question |
| --- | --- |
| Distribution | How is ... distributed? |
| Comparison | How does ... differ between ...? |
| Change | How has ... changed over ...? |
| Relationship | How do ... and ... vary together? |

For each question, state what one plotted mark should represent, compare
answers with another learner, and discuss what different marks or groupings
would make easier or harder to see.

---

## Check your understanding

1. Why does a documented table still benefit from visualization?
2. Which decisions make a chart an analytical choice rather than a neutral image?
3. How do exploratory and explanatory visualization differ?
4. Why does an unusual plotted value not prove a data error?
5. How can aggregation change the story shown by a chart?
6. Why is a visible yield-precipitation association not necessarily causal?

---

## Further resources

- [R for Data Science (2e): Data visualization](https://r4ds.hadley.nz/data-visualize.html)
  introduces the grammar of graphics using `ggplot2`.
- Claus O. Wilke, [Fundamentals of Data Visualization](https://clauswilke.com/dataviz/)
  discusses visual encodings, scales, colour, uncertainty, and figure design.
- Kieran Healy, [Data Visualization: A Practical Introduction](https://socviz.co/)
  develops reproducible visualization with applied examples.
- [The Turing Way: Data visualisation](https://book.the-turing-way.org/communication/visualisation/)
  connects visualization with accessible and reproducible research communication.
- [Data-to-Viz](https://www.data-to-viz.com/) provides a question-oriented
  overview of common chart families and their limitations.

---

## Continue to Concepts

Continue with [Understand data-visualization
concepts](07_02_data_visualization_concepts.md). It explains how questions, variables,
marks, mappings, scales, grouping, accessibility, and reproducible export work
together.
