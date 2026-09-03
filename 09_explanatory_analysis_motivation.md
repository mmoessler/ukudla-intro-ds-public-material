# Why explanatory modeling requires causal reasoning

---

- Last Update: 2026-08-25
- Source: [09_explanatory_analysis_motivation.md](/learning-modules/intro-ds-module/09_explanatory_analysis_motivation.md)
- Estimated reading time: 20 minutes
- Estimated activity time: 10 minutes

---

## Outline

- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Description is not explanation](#description-is-not-explanation)
- [A regression coefficient does not create causality](#a-regression-coefficient-does-not-create-causality)
- [Causal questions compare potential outcomes](#causal-questions-compare-potential-outcomes)
- [Research design comes before estimation](#research-design-comes-before-estimation)
- [The precipitation intervention is difficult to define](#the-precipitation-intervention-is-difficult-to-define)
- [What can go wrong](#what-can-go-wrong)
- [How this connects to the maize-yield project](#how-this-connects-to-the-maize-yield-project)
- [Initial activity](#initial-activity)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- distinguish descriptive association from causal explanation;
- explain why regression adjustment alone does not identify a causal effect;
- formulate a causal question using a target population, exposure contrast, outcome, and time horizon;
- explain why causal analysis requires untestable assumptions and domain knowledge;
- identify ambiguities in a proposed precipitation intervention; and
- describe what the maize data can contribute even when a causal effect is not identified.

---

## Place in the session

This is the **Motivation** part of the Explanatory Modeling session:

~~~text
Motivation  →  Concepts  →  Application
    ↑
 this page
~~~

Descriptive Data Analysis quantified distributions, temporal changes, and
yield-precipitation associations. It deliberately avoided causal language.
This session now asks whether and under which assumptions an observed
relationship can support a causal explanation.

[Understand explanatory-modeling concepts](09_explanatory_analysis_concepts.md)
separates causal questions, identification, and estimation. [Conduct a causal
analysis of maize yield](09_explanatory_analysis_application.md) applies that
framework to the example project.

---

## Description is not explanation

A descriptive analysis can establish that two variables vary together in the
observed data. For example, it can report that:

- maize yield differs between wetter and drier country-years;
- the correlation differs across countries or periods;
- yield levels and trends differ strongly between countries; or
- the pooled association differs from within-country associations.

These are useful scientific findings, but they do not answer what would happen
if precipitation were changed while other relevant conditions were held
appropriately comparable.

Several processes can produce an observed association:

- precipitation may affect plant growth and therefore yield;
- temperature and atmospheric conditions may affect both rainfall and yield;
- countries may differ in climate, soils, irrigation, inputs, varieties, and reporting;
- both precipitation and yield may change over time;
- measurement and aggregation may distort the relationship; or
- selection into the observed dataset may create a comparison that differs from the target population.

Explanatory modeling must distinguish these possibilities through a causal
question, a defensible research design, explicit assumptions, and an estimator
that matches them.

---

## A regression coefficient does not create causality

Linear regression can describe how an outcome differs with an explanatory
variable while holding included variables constant in the model. This is a
conditional association.

It becomes a causal effect only if additional conditions justify interpreting
the modeled comparison as the relevant counterfactual comparison. Adding
country indicators, a time trend, or many available variables does not
automatically satisfy those conditions.

An adjustment variable can help, do nothing, or introduce bias. The result
depends on its causal role:

- a **confounder** is a common cause of exposure and outcome and may need adjustment;
- a **mediator** lies on the causal pathway and should not be adjusted for when estimating a total effect;
- a **collider** is caused by two other variables and conditioning on it can open a biased path;
- a **proxy** may only partially represent an unmeasured causal factor; and
- a **precision variable** predicts the outcome without removing confounding.

These roles cannot be learned from a regression table alone. They require
temporal ordering, substantive knowledge, and an explicit causal model.

---

## Causal questions compare potential outcomes

A causal effect compares outcomes under alternative conditions for the same
target units. Let \(Y_i(p)\) denote the potential maize yield for unit \(i\)
under precipitation exposure \(p\). A unit-level contrast might be:

\[
Y_i(p + 100) - Y_i(p)
\]

Only one of these potential outcomes can be observed for a particular
country-year. The alternative is counterfactual. Causal inference therefore
uses observed outcomes from other units or times to construct a valid
comparison under assumptions.

A useful causal question names:

- **target population:** which countries, years, farms, or fields;
- **exposure:** the condition or intervention being contrasted;
- **comparison:** the alternative exposure level or policy;
- **outcome:** the measure and time at which it is evaluated;
- **estimand:** the population-level causal quantity of interest; and
- **time zero and follow-up:** when exposure assignment and outcome measurement begin.

Without these elements, “the effect of rainfall on yield” is too ambiguous for
one model coefficient to answer.

---

## Research design comes before estimation

The order of a causal analysis should be:

~~~text
causal question and target population
                 ↓
exposure contrast and causal estimand
                 ↓
causal structure and identification assumptions
                 ↓
observed-data representation and adjustment strategy
                 ↓
estimator, uncertainty, diagnostics, and sensitivity analysis
                 ↓
conclusion bounded by the design and assumptions
~~~

Beginning with a regression formula reverses this logic. It encourages the
analyst to interpret whichever coefficient software returns rather than to
ask whether that coefficient estimates the intended effect.

A model can fit the observed data well while the causal design remains
invalid. Conversely, an honest causal analysis may conclude that the available
data do not identify the intended effect. That conclusion exposes which
measurements or design changes are needed and is more useful than an
unsupported causal claim.

---

## The precipitation intervention is difficult to define

The example project records October-April country-area precipitation totals.
A proposed contrast of “100 mm more precipitation” leaves several questions:

- Does the additional precipitation fall early, late, or evenly through the season?
- Does it arrive as useful moderate rain or damaging extreme events?
- Does it fall over maize-growing land or elsewhere in the country?
- Is the contrast natural weather variation, irrigation, or another intervention?
- Does additional rain also change flooding, soil moisture, disease, or input decisions?
- Is the same contrast meaningful in every country and baseline climate?

Different versions can produce different yield outcomes. This is a
**consistency** problem: one numeric exposure value may not correspond to one
well-defined treatment condition.

The exposure is also spatially aggregated. National yield and country-area
precipitation do not show whether maize fields received the recorded rain.
Better causal analysis would benefit from subnational crop locations,
phenology, temperature, irrigation, soils, management, and field-level or
remote-sensing measurements aligned to the relevant growing season.

---

## What can go wrong

### Statistical significance is mistaken for causality

A small p-value is calculated under a statistical model. It does not establish
exchangeability, correct measurement, or a well-defined intervention.

### Every available variable is treated as a control

Data-driven adjustment can include mediators or colliders while omitting
important common causes. Select variables from a causal model, not only from
availability or association strength.

### Country and year indicators are called complete adjustment

They can account for some stable country differences and common temporal
patterns. They cannot measure every changing agricultural, economic, or
weather-related confounder.

### Model diagnostics are treated as causal diagnostics

Residual plots can reveal functional-form and variance problems. They cannot
show that unmeasured confounding is absent.

### Precision hides identification uncertainty

A narrow confidence interval can coexist with severe unmeasured confounding,
measurement error, interference, or exposure ambiguity.

---

## How this connects to the maize-yield project

The project provides 297 observations for nine countries from 1990 through
2022. Each row combines national maize yield with a country-area seasonal
precipitation estimate.

This creates an instructive but limited causal-analysis case. Learners can:

1. define a provisional precipitation-yield estimand;
2. draw a causal diagram and inventory plausible causes;
3. evaluate consistency, exchangeability, positivity, interference, and measurement;
4. compare unadjusted, country-adjusted, time-adjusted, and combined regressions;
5. inspect overlap, residuals, influential observations, and temporal structure;
6. examine sensitivity to linearity and model specification; and
7. decide which interpretation the evidence supports.

The likely defensible result is an adjusted association accompanied by a clear
statement that the current dataset does not identify a causal precipitation
effect. The causal analysis remains valuable because it makes this boundary
and the required next data explicit.

---

## Initial activity

Complete this causal-question template before reading the Concepts page:

| Element | Provisional definition | Remaining ambiguity |
| --- | --- | --- |
| Target population |  |  |
| Exposure |  |  |
| Comparison exposure |  |  |
| Outcome |  |  |
| Time zero and follow-up |  |  |
| Average causal effect |  |  |

Then list three variables that may cause both precipitation exposure and maize
yield. For each one, state whether the project measures it adequately.

---

## Check your understanding

1. How does a descriptive association differ from a causal effect?
2. Why does adding variables to a regression not automatically remove bias?
3. What elements make a causal question sufficiently precise?
4. Why can only one potential outcome be observed for a unit?
5. Why is “100 mm more precipitation” not necessarily one treatment?
6. What can regression diagnostics assess, and what can they not assess?
7. Why can failure to identify a causal effect be a useful scientific conclusion?

---

## Further resources

- Miguel A. Hernán and James M. Robins,
  [Causal Inference: What If](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/)
  develops causal questions, potential outcomes, identification, and estimation.
- Scott Cunningham, [Causal Inference: The Mixtape](https://mixtape.scunning.com/)
  introduces causal designs and their assumptions through applied examples.
- Matheus Facure, [Causal Inference for the Brave and True](https://matheusfacure.github.io/python-causality-handbook/)
  provides an accessible applied introduction.
- [DAGitty](https://www.dagitty.net/) supports drawing and evaluating causal diagrams.
- [The Turing Way: Reproducible Research](https://book.the-turing-way.org/reproducible-research/)
  provides guidance for transparent, reviewable research workflows.

---

## Continue to Concepts

Continue with [Understand explanatory-modeling
concepts](09_explanatory_analysis_concepts.md). It distinguishes causal
estimands, identification assumptions, regression estimators, uncertainty,
diagnostics, and sensitivity analysis.
