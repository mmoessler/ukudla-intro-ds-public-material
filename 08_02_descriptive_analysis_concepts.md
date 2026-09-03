# 8.2) Understand descriptive-data-analysis concepts

---

- Last Update: 2026-08-25
- Source: [08_02_descriptive_analysis_concepts.md](/learning-modules/intro-ds-module/08_02_descriptive_analysis_concepts.md)
- Estimated reading time: 60 minutes
- Estimated activity time: 30 minutes

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Start with a descriptive contract](#start-with-a-descriptive-contract)
- [Define population, sample, and observation grain](#define-population-sample-and-observation-grain)
- [Report coverage and missingness](#report-coverage-and-missingness)
- [Describe location](#describe-location)
  - [Arithmetic mean](#arithmetic-mean)
  - [Median and quantiles](#median-and-quantiles)
- [Describe dispersion](#describe-dispersion)
  - [Range and interquartile range](#range-and-interquartile-range)
  - [Variance and standard deviation](#variance-and-standard-deviation)
- [Describe distribution shape](#describe-distribution-shape)
- [Compare groups and periods deliberately](#compare-groups-and-periods-deliberately)
- [Quantify association](#quantify-association)
- [Separate pooled and within-group association](#separate-pooled-and-within-group-association)
- [Account for ordered observations](#account-for-ordered-observations)
- [Understand stationarity](#understand-stationarity)
- [Diagnose change descriptively](#diagnose-change-descriptively)
- [Treat formal tests as optional evidence](#treat-formal-tests-as-optional-evidence)
- [Connect description to modeling decisions](#connect-description-to-modeling-decisions)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
  - [Descriptive analysis and association](#descriptive-analysis-and-association)
  - [Time and stationarity](#time-and-stationarity)
  - [Reproducible interpretation](#reproducible-interpretation)
- [Continue to Application](#continue-to-application)

---

## Learning objectives

After completing this page, you should be able to:

- define the population, sample, grain, grouping, period, and denominator of a summary;
- report coverage and missingness alongside descriptive measures;
- select complementary measures of location, dispersion, and shape;
- distinguish covariance from correlation and association from causation;
- explain why pooled and within-group associations can differ;
- define stationarity and weak stationarity at an introductory level;
- identify evidence of trends, changing variance, structural change, seasonality, and association drift; and
- translate descriptive findings into requirements for later models.

---

## Place in the session

This is the **Concepts** part of the Descriptive Data Analysis session:

~~~text
Motivation  →  Concepts  →  Application
                ↑
             this page
~~~

[Why describe data numerically?](08_01_descriptive_analysis_motivation.md) explains why
precise summaries and temporal stability matter. This page provides the tools
used in [the maize descriptive-analysis
application](08_03_descriptive_analysis_application.md).

Use one central question throughout:

> Which observations does this statistic describe, which feature does it
> summarize, and what important structure can it conceal?

---

## Start with a descriptive contract

Define a summary before calculating it:

| Contract element | Question |
| --- | --- |
| Analytical question | Which feature or comparison should be quantified? |
| Population and sample | About which observations is the statement intended? Which were observed? |
| Grain and key | What does one row represent, and what identifies it? |
| Variable and unit | What is measured and in which unit? |
| Group and period | Which observations are summarized together? |
| Missing-value rule | Which values are unavailable, and how are they handled? |
| Measure | Why is this statistic appropriate? |
| Companion evidence | Which count, alternative measure, or graphic prevents misinterpretation? |
| Claim boundary | What does the result not establish? |

This contract prevents a software function from defining the question by
accident. It also makes outputs reviewable and comparable across analysts.

---

## Define population, sample, and observation grain

The **target population** is the set of units or events about which a
statement is intended; the **observed sample** is the subset represented in
the data. A census-like administrative dataset can still cover only selected
variables, periods, or reporting units.

The **observation grain** states what one row represents. In the integrated
maize data, one row represents one selected country and one year, with a
growing season ending in that year — not an individual farm, field,
household, or weather station.

The grain determines valid denominators and weights. An unweighted mean
across country-years gives every available country-year the same influence;
it does not weight by harvested area, production, population, or country
size. Such a mean can be correct for one question and inappropriate for
another.

---

## Report coverage and missingness

Begin every descriptive table with coverage: total rows considered,
non-missing and missing values, number of groups, first and last observed
period, and gaps or duplicated keys.

Let \(n\) be the number of non-missing observations used by a statistic.
Report it even if the full dataset has a known row count — pairing two
variables can reduce the effective \(n\) because both values must be present.

Missingness can itself vary by group or time, so a changing mean may partly
reflect changing coverage rather than changing values. Describing only
complete cases is insufficient when excluded observations are systematic.

---

## Describe location

Measures of location describe a distribution's centre or typical value.

### Arithmetic mean

For values \(x_1, \ldots, x_n\), the sample mean is:

\[
\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i
\]

The mean uses every value and supports many later methods, but is sensitive
to extreme values, skewness, and weighting choices.

### Median and quantiles

The median is the middle ordered value, or the average of the two middle
values for an even count. It resists extremes but does not use the
magnitude of every deviation. The \(p\)-quantile divides ordered observations
so that approximately a proportion \(p\) lies at or below it; the first and
third quartiles correspond to 25% and 75% (algorithms can differ slightly
for small samples, so use a consistent implementation).

Report mean and median together when skewness or unusual observations may
matter — their difference is a diagnostic, not a full measure of shape.

---

## Describe dispersion

Dispersion measures how widely observations vary.

### Range and interquartile range

The range is the maximum minus the minimum: easy to interpret, but it depends
entirely on two observations and usually grows with sample size. The
interquartile range describes the middle half of values,
\(IQR = Q_{0.75} - Q_{0.25}\), and is resistant to extremes.

### Variance and standard deviation

The sample variance is:

\[
s^2 = \frac{1}{n-1}\sum_{i=1}^{n}(x_i - \bar{x})^2
\]

The standard deviation is \(s = \sqrt{s^2}\), returning to the variable's
unit. Both are sensitive to extreme values; standard deviation is not the
average absolute distance from the mean and does not define a universal
interval containing a fixed percentage of observations. Pair standard
deviation with the mean and IQR with the median — both pairs help when a
distribution is asymmetric or heavy-tailed.

---

## Describe distribution shape

Location and dispersion cannot uniquely identify a distribution. Also
inspect symmetry or skewness (whether one tail extends further), modality
(whether values form one or several concentrations), boundaries (whether
measurement or definition imposes limits), tails (whether extreme values
occur frequently), and gaps or heaping (whether rounding, thresholds, or
missing regions appear).

Numerical skewness and kurtosis exist, but plots and quantiles are often more
interpretable for an introductory analysis. An observation beyond a boxplot
whisker is not automatically an error — it is selected by a rule for further
investigation.

---

## Compare groups and periods deliberately

Group summaries expose structure hidden by pooled statistics. Use the same
variables, units, missing-value rule, and measures across groups, and always
report group-specific counts.

Period definitions should follow the analytical workflow rather than be
chosen after seeing an attractive contrast. The maize project's modeling
workflow already distinguishes 1990–2005 (earlier history), 2006–2017
(recent training history), and 2018–2022 (later test period). This split
supports a descriptive check of whether the period reserved for later
evaluation resembles the preceding training period. Differences can be
absolute, percentage-based, or standardized, but every choice needs an
interpretable denominator; avoid percentage change when the reference is
zero or near zero.

Description alone does not determine whether a difference is substantively
important. Compare magnitude with units, variation, domain knowledge, and the
consequences of a decision.

---

## Quantify association

For paired observations \((x_i, y_i)\), sample covariance is:

\[
s_{xy} = \frac{1}{n-1}\sum_{i=1}^{n}(x_i-\bar{x})(y_i-\bar{y})
\]

Its sign gives the direction of linear co-variation; its magnitude depends on
both variables' units, making comparisons across scales difficult.

Pearson correlation standardizes covariance:

\[
r = \frac{s_{xy}}{s_x s_y}
\]

It lies between -1 and 1 when both standard deviations are positive. Values
near -1 or 1 indicate strong negative or positive *linear* association; a
value near zero indicates weak linear association, not necessarily no
relationship.

Correlation can be unstable with small samples and sensitive to outliers. It
must be reported with paired \(n\) and inspected with a scatterplot. It does
not establish a causal effect, and its square is not automatically the
proportion of variance explained outside a specified model.

---

## Separate pooled and within-group association

A pooled correlation combines within-country variation with differences
between country means. It may answer a different question from correlations
calculated separately for each country.

Countries with higher average precipitation may also have higher average
yield, producing a positive pooled relationship even if wetter years within
each country are not consistently higher-yielding — or the reverse. This is
related to aggregation effects and Simpson's paradox.

Compare the pooled scatterplot and correlation against country-faceted
plots and country-specific correlations, period-specific correlations, and,
if appropriate, deviations from country-specific means. These are
descriptive views, not substitutes for a model that explicitly represents
country and time.

---

## Account for ordered observations

Country-year observations are ordered in time. Adjacent years can be more
similar than distant years, so observations may not be independent.

**Autocovariance** and **autocorrelation** quantify association between a
series and lagged versions of itself — a lag-one autocorrelation compares
values one period apart, and its interpretation requires a regular time
order and care around gaps.

Temporal dependence matters because the effective information can be less
than the row count suggests, and it explains why shuffling observations into
random training and test sets can leak temporal information and produce an
unrealistic evaluation.

---

## Understand stationarity

Strict stationarity means that the joint probability distribution of a process
is unchanged by shifting time. That definition is demanding and cannot be
established through a few summaries.

**Weak stationarity** is more limited. A process \(X_t\) is weakly stationary
when:

1. its mean \(E[X_t]\) is constant over time;
2. its variance \(Var(X_t)\) is finite and constant over time; and
3. its covariance \(Cov(X_t, X_{t-k})\) depends on lag \(k\), not calendar
   time \(t\).

Stationarity is a property of a data-generating process, not merely a table:
a finite series can provide evidence consistent or inconsistent with it, but
cannot prove universal stability. Different variables or groups can have
different stability — yield may display a trend while deviations from that
trend are more stable, and one country may show a structural change that
another does not. State exactly which series, period, and property are
under discussion.

---

## Diagnose change descriptively

Look for several kinds of evidence:

| Pattern | Descriptive evidence | Why it matters |
| --- | --- | --- |
| Trend | Period means, medians, and time plots change | Constant mean may be implausible |
| Changing variance | Period SDs, IQRs, ranges, or residual spread differ | Constant variance may be implausible |
| Structural change | Abrupt persistent level or variability shift | One process may not describe all periods |
| Seasonality or cycle | Repeated pattern at meaningful lags | Dependence changes with cycle position |
| Association drift | Correlations or slopes differ by period or group | A learned relationship may not transfer |
| Coverage drift | Missingness or measurement practice changes | Apparent change may reflect observation change |

No single summary establishes stationarity — compare complementary
statistics and plots across periods selected for substantive or workflow
reasons, and record possible explanations from metadata and domain
knowledge.

---

## Treat formal tests as optional evidence

Tests such as the Augmented Dickey-Fuller or KPSS test use different null
hypotheses and assumptions; results depend on trend terms, lag choices,
sample size, structural breaks, and the variable tested, and failure to
reject a null is not proof that it is true.

For this introductory session, formal testing is optional. The core task is
to visualize each country series, compare location and dispersion across
meaningful periods, inspect whether associations change, and describe
consequences for later analysis. If a formal test is added as an extension,
report its null hypothesis, specification, limitations, and relationship to
the descriptive evidence.

---

## Connect description to modeling decisions

Descriptive findings should produce an explicit handoff:

| Finding | Possible later response |
| --- | --- |
| Countries differ strongly in level | Represent country effects or stratify |
| Yield changes over time | Include time or trend; avoid assuming a constant mean |
| Variability differs by country or period | Review transformations or group-specific uncertainty |
| Yield-precipitation association differs by country | Consider interactions or partial pooling |
| Later period differs from training history | Use a time-aware split and qualify transferability |
| Observations are temporally dependent | Avoid independence assumptions and random leakage |

These are prompts for model design, not automatic prescriptions — later
sessions must define whether the aim is explanatory or predictive and
evaluate the relevant assumptions directly.

---

## Check your understanding

1. Which elements belong in a descriptive contract?
2. How do target population, observed sample, and row grain differ?
3. Why must every statistic include an effective observation count?
4. Why do location and dispersion not fully describe distribution shape?
5. How can a pooled correlation differ from within-country correlations?
6. Why does temporal ordering challenge an independence assumption?
7. What three properties define weak stationarity?
8. How can descriptive findings change a later train-test strategy?

---

## Further resources

### Descriptive analysis and association

- [OpenIntro Statistics](https://www.openintro.org/book/os/)
- [R for Data Science (2e): Exploratory data analysis](https://r4ds.hadley.nz/EDA.html)
- [NIST/SEMATECH e-Handbook: Exploratory Data Analysis](https://www.itl.nist.gov/div898/handbook/eda/eda.htm)

### Time and stationarity

- [Forecasting: Principles and Practice — Time-series features](https://otexts.com/fpp3/features.html)
- [Forecasting: Principles and Practice — Stationarity and differencing](https://otexts.com/fpp3/stationarity.html)

### Reproducible interpretation

- [The Turing Way: Research Data Management](https://book.the-turing-way.org/reproducible-research/rdm/)
- [The Turing Way: Data visualisation](https://book.the-turing-way.org/communication/visualisation/)

---

## Continue to Application

Continue with [Describe maize yield and
precipitation](08_03_descriptive_analysis_application.md). You will define descriptive
contracts, generate coverage and summary tables, compare periods and groups,
quantify associations, assess evidence of change, and write a modeling handoff.
