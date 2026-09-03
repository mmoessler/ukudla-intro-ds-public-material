# 9.1) Why explanatory modeling requires causal reasoning

---

- Last Update: 2026-09-03
- Source: [09_01_explanatory_analysis_motivation.md](/learning-modules/intro-ds-module/09_01_explanatory_analysis_motivation.md)
- Estimated reading time: 20 minutes
- Estimated activity time: 10 minutes

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Description is not explanation](#description-is-not-explanation)
- [A regression coefficient does not create causality](#a-regression-coefficient-does-not-create-causality)
- [Causal questions compare potential outcomes](#causal-questions-compare-potential-outcomes)
- [Research design comes before estimation](#research-design-comes-before-estimation)
- [Study design changes what can be explained](#study-design-changes-what-can-be-explained)
- [The precipitation intervention is difficult to define](#the-precipitation-intervention-is-difficult-to-define)
- [What can go wrong](#what-can-go-wrong)
- [Worked example: the maize-yield project](#worked-example-the-maize-yield-project)
- [Initial activity](#initial-activity)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- distinguish descriptive association from causal explanation;
- explain why regression adjustment alone does not identify a causal effect;
- formulate a causal question with a target population, exposure contrast, outcome, and time horizon;
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

Descriptive Data Analysis quantified distributions, changes, and associations,
deliberately avoiding causal language. This session asks whether and under
which assumptions a comparison can support a causal explanation.

[Concepts](09_02_explanatory_analysis_concepts.md) separates causal questions,
identification, and estimation across experimental and observational designs.
The [application](09_03_explanatory_analysis_application.md) uses an aggregated
observational example to show why causal conclusions sometimes remain limited.

---

## Description is not explanation

A descriptive analysis can show that outcomes differ between observed
treatment, exposure, site, batch, or period groups. These findings do not by
themselves answer what would happen to the same target population under an
alternative intervention.

Several distinct processes can produce the same observed association: a real
effect, common causes, treatment selection, batch or site differences,
measurement error, aggregation, attrition, or selection into the dataset.

Explanatory modeling must distinguish these possibilities through a causal
question, a defensible research design, explicit assumptions, and a matching
estimator.

---

## A regression coefficient does not create causality

Linear regression describes how an outcome differs with an explanatory
variable while holding included variables constant — a conditional
association, not automatically a causal effect. It becomes causal only if
additional conditions justify treating the modeled comparison as the relevant
counterfactual. Adding country indicators, a time trend, or many available
variables does not automatically satisfy those conditions.

An adjustment variable can help, do nothing, or introduce bias, depending on
its causal role:

- a **confounder** is a common cause of exposure and outcome and may need adjustment;
- a **mediator** lies on the causal pathway and should not be adjusted for when estimating a total effect;
- a **collider** is caused by two other variables, and conditioning on it can open a biased path;
- a **proxy** partially represents an unmeasured causal factor; and
- a **precision variable** predicts the outcome without removing confounding.

These roles cannot be learned from a regression table alone — they require
temporal ordering, substantive knowledge, and an explicit causal model.

---

## Causal questions compare potential outcomes

A causal effect compares outcomes under alternative conditions for the same
target units. Let \(Y_i(p)\) denote the potential maize yield for unit \(i\)
under precipitation exposure \(p\). A unit-level contrast might be:

\[
Y_i(p + 100) - Y_i(p)
\]

Only one of these potential outcomes can be observed for a given
country-year — the alternative is counterfactual. Causal inference therefore
uses observed outcomes from other units or times to construct a valid
comparison under assumptions.

A useful causal question names:

- **target population:** which countries, years, farms, or fields;
- **exposure and comparison:** the contrasted conditions or policies;
- **outcome:** the measure and time at which it is evaluated;
- **estimand:** the population-level causal quantity of interest; and
- **time zero and follow-up:** when exposure and outcome measurement begin.

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

Beginning with a regression formula reverses this logic: it encourages
interpreting whichever coefficient software returns rather than asking
whether that coefficient estimates the intended effect. A model can fit the
data well while the causal design remains invalid; an honest analysis
concluding that the data do not identify the effect is more useful than an
unsupported causal claim.

---

## Study design changes what can be explained

In a laboratory or field experiment, randomized assignment can create a
credible comparison when the intervention, experimental unit, outcome timing,
and analysis preserve the design. Blocking, clustering, noncompliance,
attrition, spillover, and measurement remain relevant.

In a field observational or secondary-data study, exposure is not assigned by
the analyst. Identification then requires an explicit account of why exposed
and comparison units differ, which pre-exposure common causes are measured,
whether relevant contrasts have support, and how selection and measurement
affect the result.

The worked example deliberately illustrates a difficult observational case.

## The precipitation intervention is difficult to define

The example project records October-April country-area precipitation totals.
A proposed contrast of "100 mm more precipitation" leaves several questions
unresolved: rain timing and intensity, whether it falls over maize-growing
land, whether it reflects natural variation or irrigation, and whether the
same contrast is meaningful in every country and baseline climate.

Different versions can produce different yield outcomes. This is a
**consistency** problem: one numeric exposure value may not correspond to one
well-defined treatment condition.

The exposure is also spatially aggregated: national yield and country-area
precipitation do not show whether maize fields actually received the
recorded rain. A stronger analysis would need field-level or remote-sensing
measurements aligned to the growing season.

---

## What can go wrong

- **Significance mistaken for causality:** a small p-value does not establish exchangeability, correct measurement, or a well-defined intervention.
- **Every available variable treated as a control:** data-driven adjustment can include mediators or colliders while omitting important common causes.
- **Country/year indicators called complete adjustment:** they capture stable country differences and common trends, not every changing confounder.
- **Diagnostics treated as causal diagnostics:** residual plots reveal functional-form problems, not whether unmeasured confounding is absent.
- **Precision mistaken for identification:** a narrow confidence interval can coexist with severe confounding, measurement error, or exposure ambiguity.

---

## Worked example: the maize-yield project

The project provides 297 observations for nine countries from 1990 through
2022, each combining national maize yield with a country-area seasonal
precipitation estimate — an instructive but limited causal-analysis case.
Learners will:

1. define a provisional precipitation-yield estimand and draw a causal diagram;
2. evaluate consistency, exchangeability, positivity, interference, and measurement;
3. compare unadjusted, country-adjusted, time-adjusted, and combined regressions;
4. inspect residuals, influential observations, and functional-form sensitivity; and
5. decide which interpretation the evidence supports.

The likely defensible result is an adjusted association together with a clear
statement that the dataset does not identify a causal precipitation effect —
valuable because it makes this boundary and the required next data explicit.

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

Then list three variables that may cause both precipitation exposure and
yield, and state whether the project measures each one adequately.

---

## Check your understanding

1. How does a descriptive association differ from a causal effect?
2. Why does adding variables to a regression not automatically remove bias?
3. What elements make a causal question sufficiently precise?
4. Why is "100 mm more precipitation" not necessarily one treatment?
5. Why can failure to identify a causal effect be a useful scientific conclusion?

---

## Further resources

- Hernán and Robins, [Causal Inference: What If](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/) — causal questions, potential outcomes, identification, and estimation.
- Cunningham, [Causal Inference: The Mixtape](https://mixtape.scunning.com/) — causal designs and assumptions through applied examples.
- Facure, [Causal Inference for the Brave and True](https://matheusfacure.github.io/python-causality-handbook/) — an accessible applied introduction.
- [DAGitty](https://www.dagitty.net/) — drawing and evaluating causal diagrams.
- [The Turing Way: Reproducible Research](https://book.the-turing-way.org/reproducible-research/) — transparent, reviewable research workflows.

---

## Continue to Concepts

Continue with [Understand explanatory-modeling
concepts](09_02_explanatory_analysis_concepts.md). It distinguishes causal
estimands, identification assumptions, regression estimators, uncertainty,
diagnostics, and sensitivity analysis.
