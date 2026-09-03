# 8.1) Why describe data numerically?

---

- Last Update: 2026-08-25
- Source: [08_01_descriptive_analysis_motivation.md](/learning-modules/intro-ds-module/08_01_descriptive_analysis_motivation.md)
- Estimated reading time: 20 minutes
- Estimated activity time: 10 minutes

---

## Outline

- [Outline](#outline)
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

[Understand descriptive-data-analysis concepts](08_02_descriptive_analysis_concepts.md)
develops the required measures and the idea of stationarity. [Describe maize
yield and precipitation](08_03_descriptive_analysis_application.md) applies them in the
example project.

---

## Visual patterns need numerical descriptions

A plot can make a pattern visible, but readers may still need precise
answers: how many observations support it, what a typical value is, how
strongly values vary, how asymmetric the distribution is, how different
countries or periods are, and how strongly two variables vary together.

Numerical summaries make such comparisons explicit and reproducible, and can
reveal information that is hard to judge visually, such as an exact median,
an interquartile range, or the number of missing observations.

~~~text
visualization: reveal structure and exceptions
description:   quantify selected features of that structure
interpretation: connect both to scope, assumptions, and limitations
~~~

Neither is sufficient by itself. Different distributions can share a mean and
standard deviation, while the same data can look different under a changed
visual scale. Inspect observations and calculate relevant summaries.

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

The right combination depends on the question and distribution: the mean
uses every value but is sensitive to extremes; the median resists extremes
but says nothing about variability; standard deviation pairs naturally with
the mean but can mislead for skewed data; quantiles are robust but omit
detail. The objective is a small complementary set that answers a stated
question, not every statistic software makes available.

---

## Descriptions depend on scope

A statistic describes the observations that entered its calculation. Its
meaning changes with the target population and observed sample, the unit
represented by one row, filters and inclusion rules, groups and time
periods, weighting choices, and the treatment of missing values.

An overall mean across all country-years is not the mean for a typical
country unless each contributes equally and that weighting matches the
question; a mean national yield is not the yield of a typical farm; and a
correlation pooled across countries can differ from correlations calculated
within individual countries.

Always report an effective sample size, and state how many observations
remain after removing missing values. A precisely calculated statistic with
unclear scope is not a meaningful result.

---

## Data-generating processes can change

Many introductory examples treat observations as if they came from an
unchanging process, but real food-system data can violate that assumption.
Yields may change because of technology, seed varieties, irrigation, input
access, policy, reporting practices, land use, or climate; precipitation can
display cycles, persistent anomalies, or changing variability. A later
period may show a different typical yield, wider or narrower variation, a
changed distribution shape, a structural break, or a changed
yield-precipitation association. These changes are part of the description,
not nuisances to hide, and may determine whether historical evidence is
relevant for a later period.

---

## Why stationarity matters before modeling

**Stationarity** concerns whether the probabilistic properties of a process
remain stable over time — whether level, variation, distribution, and
temporal dependence appear comparable across the period being studied. It
matters because later explanatory and predictive models learn relationships
from observed data: if the process changes, a historical average may poorly
represent a recent year, a relationship may not transfer across periods, a
random train-test split can mix past and future and exaggerate performance,
and a model may need explicit time, trend, season, or group terms.

Descriptive analysis cannot prove stationarity from a finite dataset, and
non-stationarity does not make analysis impossible. The practical goal is to
look for evidence of change, document it, and carry it into the modeling
strategy — formal tests can help later but should not replace plots, subject
knowledge, and explicit period comparisons.

---

## What can go wrong

Common misreadings of a descriptive summary include pooled statistics that
hide group differences (report group-specific summaries when groups matter),
missing values dropped silently by default functions (report the effective
observation count), correlation read as a causal claim (it quantifies linear
co-variation only), and a trend mistaken for a stable relationship (two
variables can correlate merely because both change over time). A formal
stationarity test does not resolve these either — it depends on assumptions,
specification, and a chosen null hypothesis, so its result is evidence
within that setup, not a universal verdict.

---

## How this connects to the maize-yield project

The example project contains 297 country-year observations for nine countries
from 1990 through 2022, combining national maize yield (tonnes per hectare)
and CHIRPS October-April country-area precipitation (millimetres).

The visualization session showed distributions, country trajectories, and
paired yield-precipitation observations. This session quantifies them:
confirming population, grain, coverage, and missingness; summarizing yield
and precipitation location and dispersion by country and period (early,
recent training, later test); comparing pooled and country-specific
associations; and documenting evidence for or against stability over time.

The result is not an explanatory or predictive model, but an evidence base
for deciding what such a model must represent and how it should be evaluated.

---

## Initial activity

Return to the maize-yield trend and yield-precipitation figures and propose
one statistic and one complementary graphic per question:

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
4. What kinds of change provide evidence against stationarity?
5. Why can a finite dataset not prove that a process is stationary?

---

## Further resources

- [OpenIntro Statistics](https://www.openintro.org/book/os/) — distributions,
  summary measures, and statistical reasoning.
- [R for Data Science (2e): Exploratory data analysis](https://r4ds.hadley.nz/EDA.html)
  — questions, variation, and covariation in R.
- [Forecasting: Principles and Practice — Stationarity](https://otexts.com/fpp3/stationarity.html)
  — an accessible introduction to time-series stationarity.
- [The Turing Way: Research Data Management](https://book.the-turing-way.org/reproducible-research/rdm/)
  — documented, reproducible analysis practice.
- [NIST/SEMATECH e-Handbook of Statistical Methods](https://www.itl.nist.gov/div898/handbook/)
  — a broad reference for exploratory statistics.

---

## Continue to Concepts

Continue with [Understand descriptive-data-analysis
concepts](08_02_descriptive_analysis_concepts.md). It explains observation scope, coverage,
location, dispersion, shape, association, temporal dependence, and
stationarity.
