# Why describe data numerically?

---

- Last Update: 2026-08-25
- Source: [08_descriptive_analysis_motivation.md](/learning-modules/intro-ds-module/08_descriptive_analysis_motivation.md)
- Estimated reading time: 20 minutes
- Estimated activity time: 10 minutes

---

## Outline

- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Visual patterns need numerical descriptions](#visual-patterns-need-numerical-descriptions)
- [A single average is not a description](#a-single-average-is-not-a-description)
- [Descriptions depend on scope](#descriptions-depend-on-scope)
- [Data-generating processes can change](#data-generating-processes-can-change)
- [Why stationarity matters before modeling](#why-stationarity-matters-before-modeling)
- [What can go wrong](#what-can-go-wrong)
- [How this connects to the maize-yield project](#how-this-connects-to-the-maize-yield-project)
- [Initial activity](#initial-activity)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- explain how numerical descriptions complement visualization;
- identify why sample size, population, grouping, period, and missingness must accompany a statistic;
- explain why location, dispersion, and distribution shape belong together;
- distinguish a stable-looking series from a claim that its process is stationary;
- describe why changing levels, variation, or associations matter for later models; and
- formulate descriptive questions for the maize-yield and precipitation data.

---

## Place in the session

This is the **Motivation** part of the Descriptive Data Analysis session:

~~~text
Motivation  →  Concepts  →  Application
    ↑
 this page
~~~

Data Visualization made distributions, country differences, temporal change,
and possible yield-precipitation relationships visible. Descriptive Data
Analysis now asks how to quantify those patterns without reducing them to a
misleading single number.

[Understand descriptive-data-analysis concepts](08_descriptive_analysis_concepts.md)
develops the required measures and the idea of stationarity. [Describe maize
yield and precipitation](08_descriptive_analysis_application.md) applies them in the
example project.

---

## Visual patterns need numerical descriptions

A plot can make a pattern visible, but readers may still need precise answers:

- How many observations support the pattern?
- What is a typical value?
- How strongly do values vary?
- How asymmetric is the distribution?
- How different are countries or periods?
- How strongly do two variables vary together?

Numerical summaries make such comparisons explicit and reproducible. They can
also reveal information that is difficult to judge visually, such as an exact
median, an interquartile range, or the number of missing observations.

Visualization and description therefore complement one another:

~~~text
visualization: reveal structure and exceptions
description:   quantify selected features of that structure
interpretation: connect both to scope, assumptions, and limitations
~~~

Neither is sufficient by itself. Different distributions can have the same
mean and standard deviation, while the same data can look different under a
changed visual scale. Inspect observations and calculate relevant summaries.

---

## A single average is not a description

Suppose two countries have the same mean maize yield. One may have relatively
stable annual values; the other may alternate between very low and very high
values. The common mean conceals a difference that matters to farmers,
planners, and later models.

A useful description normally combines:

- **coverage:** observation count, time range, and missingness;
- **location:** mean, median, or selected quantiles;
- **dispersion:** standard deviation, interquartile range, or range; and
- **shape and exceptions:** skewness, boundaries, clusters, and unusual values.

The appropriate combination depends on the question and distribution. The
mean uses every value but can be strongly affected by extremes. The median is
more resistant but does not describe variability. Standard deviation relates
directly to the mean but can be unrepresentative for skewed data. Quantiles
are robust and interpretable, but also omit detail.

The objective is not to calculate every statistic available in software. It
is to select a small complementary set that answers a stated question.

---

## Descriptions depend on scope

A statistic describes the observations that entered its calculation. Its
meaning changes with:

- the target population and observed sample;
- the unit represented by one row;
- filters and inclusion rules;
- groups and time periods;
- weighting choices; and
- treatment of missing values.

An overall mean across all country-years is not the mean for a typical country
unless each country contributes equally and that weighting matches the
question. A mean national yield is not the yield of a typical farm. A
correlation pooled across countries can differ from correlations calculated
within individual countries.

Always report an effective sample size with a statistic. If missing values are
removed, state how many observations remain. A precisely calculated statistic
with unclear scope is not a meaningful result.

---

## Data-generating processes can change

Many introductory examples treat observations as if they came from an
unchanging process. Real food-system data can violate that assumption.
Agricultural yields may change because of technology, seed varieties,
irrigation, input access, policy, reporting practices, land use, or climate.
Precipitation can display cycles, persistent anomalies, or changing
variability.

Compared with an earlier period, a later period may have:

- a different typical yield;
- wider or narrower variation;
- a changed distribution shape;
- a structural break;
- a different yield-precipitation association; or
- a changed pattern of missing observations.

These changes are not nuisances to hide. They are part of the description and
may determine whether historical evidence is relevant for a later period.

---

## Why stationarity matters before modeling

**Stationarity** concerns whether the probabilistic properties of a process
remain stable over time. At an introductory level, ask whether the level,
variation, distribution, and temporal dependence appear comparable across the
period being studied.

Stationarity matters because later explanatory and predictive models learn
relationships from observed data. If the process changes:

- a historical average may poorly represent a recent year;
- one relationship may not transfer across periods;
- a random train-test split can mix past and future and exaggerate performance;
- uncertainty calculated under stable conditions may be too narrow; and
- a model may require time, trend, season, group, or structural-change terms.

Descriptive analysis cannot prove stationarity from a finite dataset. Nor does
non-stationarity make analysis impossible. The practical goal is to look for
evidence of change, document it, and carry it into the modeling strategy.
Formal tests can be useful later, but should not replace plots, subject
knowledge, and explicit period comparisons.

---

## What can go wrong

### A pooled statistic hides meaningful groups

An overall average may mix countries with different levels and trajectories.
Report overall and group-specific summaries when groups matter.

### Missing values disappear silently

Many functions remove missing values when requested. A result without the
original and effective observation counts can conceal incomplete coverage.

### Correlation becomes a causal claim

Correlation quantifies linear co-variation. It does not control for country,
time, omitted variables, aggregation, or measurement error, and it does not
identify a causal effect.

### A trend is mistaken for a stable relationship

Two variables can correlate because both change over time. Conversely, a
pooled association can disappear or reverse within countries or periods.

### A test result replaces reasoning

A stationarity test depends on assumptions, specification, sample size, and a
particular null hypothesis. A result is evidence within that setup, not a
universal verdict about the data-generating process.

---

## How this connects to the maize-yield project

The example project contains 297 country-year observations for nine countries
from 1990 through 2022. Its integrated artifact combines:

- national maize yield in tonnes per hectare; and
- CHIRPS October-April country-area precipitation in millimetres.

The visualization session showed distributions, country trajectories, and
paired yield-precipitation observations. This session quantifies them by:

1. confirming population, grain, coverage, and missingness;
2. summarizing yield location and dispersion by country;
3. comparing an early period, a recent training period, and a later test period;
4. summarizing precipitation with the same discipline;
5. comparing pooled and country-specific associations; and
6. documenting evidence for or against stability over time.

The result is not an explanatory or predictive model. It is an evidence base
for deciding what such a model must represent and how it should be evaluated.

---

## Initial activity

Return to the maize-yield trend and yield-precipitation figures. For each
question, propose one statistic and one complementary graphic:

| Question | Statistic | Graphic |
| --- | --- | --- |
| What is a typical annual yield? |  |  |
| How variable is yield? |  |  |
| Did yield change between periods? |  |  |
| Do yield and precipitation vary together? |  |  |

For every proposed statistic, write its population, grouping, period, unit,
and effective observation count. Discuss what the statistic would omit.

---

## Check your understanding

1. Why should a numerical summary be inspected with a visualization?
2. Why is the mean alone an incomplete description?
3. How can grouping change the meaning of a statistic?
4. Why must missingness and effective sample size be reported?
5. What kinds of change provide evidence against stationarity?
6. Why can non-stationarity affect both explanation and prediction?
7. Why can a finite dataset not prove that a process is stationary?

---

## Further resources

- [OpenIntro Statistics](https://www.openintro.org/book/os/) introduces
  distributions, summary measures, association, and statistical reasoning.
- [R for Data Science (2e): Exploratory data analysis](https://r4ds.hadley.nz/EDA.html)
  connects questions, variation, covariation, and reproducible R workflows.
- [Forecasting: Principles and Practice — Stationarity](https://otexts.com/fpp3/stationarity.html)
  provides an accessible introduction to stationarity in time-series data.
- [The Turing Way: Research Data Management](https://book.the-turing-way.org/reproducible-research/rdm/)
  connects trustworthy analysis with documented data and reproducible practice.
- [NIST/SEMATECH e-Handbook of Statistical Methods](https://www.itl.nist.gov/div898/handbook/)
  is a broad reference for exploratory and descriptive statistical methods.

---

## Continue to Concepts

Continue with [Understand descriptive-data-analysis
concepts](08_descriptive_analysis_concepts.md). It explains observation scope, coverage,
location, dispersion, shape, association, temporal dependence, and
stationarity.
