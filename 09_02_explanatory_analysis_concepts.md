# 9.2) Understand explanatory-modeling concepts

---

- Last Update: 2026-08-25
- Source: [09_02_explanatory_analysis_concepts.md](/learning-modules/intro-ds-module/09_02_explanatory_analysis_concepts.md)
- Estimated reading time: 60 minutes
- Estimated activity time: 30 minutes

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Separate question, estimand, identification, and estimation](#separate-question-estimand-identification-and-estimation)
- [Use potential outcomes to define causal effects](#use-potential-outcomes-to-define-causal-effects)
- [Make causal structure explicit](#make-causal-structure-explicit)
- [Distinguish variable roles](#distinguish-variable-roles)
- [Evaluate identification assumptions](#evaluate-identification-assumptions)
  - [Consistency](#consistency)
  - [Exchangeability](#exchangeability)
  - [Positivity](#positivity)
  - [No interference](#no-interference)
  - [Measurement and selection](#measurement-and-selection)
- [Understand linear regression](#understand-linear-regression)
- [Interpret coefficients conditionally](#interpret-coefficients-conditionally)
- [Represent countries and time](#represent-countries-and-time)
- [Check functional form and interactions](#check-functional-form-and-interactions)
- [Interpret uncertainty carefully](#interpret-uncertainty-carefully)
- [Use diagnostics for their proper purpose](#use-diagnostics-for-their-proper-purpose)
- [Compare specifications as sensitivity evidence](#compare-specifications-as-sensitivity-evidence)
- [Bound the causal conclusion](#bound-the-causal-conclusion)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Application](#continue-to-application)

---

## Learning objectives

After completing this page, you should be able to:

- distinguish a causal question, estimand, identification strategy, estimator, and estimate;
- define an average causal effect with potential outcomes;
- use a directed acyclic graph to express causal assumptions;
- distinguish confounders, mediators, colliders, proxies, and precision variables;
- explain consistency, exchangeability, positivity, no interference, and measurement requirements;
- interpret regression coefficients as conditional associations;
- state the additional conditions required for causal interpretation;
- evaluate functional form, residuals, influence, temporal dependence, and specification sensitivity; and
- distinguish statistical uncertainty from identification uncertainty.

---

## Place in the session

This is the **Concepts** part of the Explanatory Modeling session:

~~~text
Motivation  →  Concepts  →  Application
                ↑
             this page
~~~

[Why explanatory modeling requires causal
reasoning](09_01_explanatory_analysis_motivation.md) motivates the distinction
between association and causation. This page provides the framework used in
[the maize-yield causal analysis](09_03_explanatory_analysis_application.md).

Use one organizing principle:

> A statistical model receives causal meaning from the question, design, and
> identification assumptions—not from the model family or coefficient name.

---

## Separate question, estimand, identification, and estimation

These stages answer different questions:

| Stage | Question | Maize example |
| --- | --- | --- |
| Causal question | What change and outcome should be understood? | What would happen to yield under a specified precipitation contrast? |
| Estimand | Which causal quantity summarizes the question? | Average difference in potential yield under two exposures |
| Identification | Under which assumptions can observed data express it? | Conditional exchangeability and exposure overlap |
| Estimator | Which procedure calculates the identified quantity? | Regression adjustment under a stated model |
| Estimate | What value resulted in this sample? | Yield difference per 100 mm with uncertainty |

An estimator can be computed when identification fails. It then estimates an
observed-data association, not the intended causal estimand.

---

## Use potential outcomes to define causal effects

Let \(Y_i(p)\) represent the maize yield unit \(i\) would have under
precipitation exposure \(p\). For two specified levels, the unit-level effect
is:

\[
Y_i(p_1)-Y_i(p_0)
\]

The average causal effect over a target population is:

\[
E\left[Y(p_1)-Y(p_0)\right]
\]

The same unit cannot reveal both outcomes at the same time. Causal inference
uses other observed units or times as comparisons under assumptions.

For continuous precipitation, one might target the contrast between \(p\) and
\(p+100\) mm or a dose-response function \(E[Y(p)]\). A constant linear slope
assumes the same marginal difference across the supported exposure range. That
may be implausible if drought and excessive rain both reduce yield.

---

## Make causal structure explicit

A **directed acyclic graph** (DAG) represents assumed causal relationships:

- nodes represent variables or concepts;
- arrows represent direct causal influence at the chosen abstraction;
- paths represent possible routes of association; and
- acyclic means the represented time order contains no feedback loop.

A provisional maize DAG could contain:

~~~text
weather system ─────► precipitation ─────► yield
      │                                      ▲
      └──────────────────────────────────────┘

country and time context ─► precipitation
              │                   │
              └──────────────────►yield

irrigation, inputs, soils and varieties ───► yield
~~~

The diagram is not discovered by selecting significant correlations. It is a
claim based on domain knowledge, temporality, measurement, and the research
question. Record plausible alternatives.

A broad node such as “country and time context” hides many mechanisms. Drawing
it does not mean a country indicator and year term measure every relevant
cause.

---

## Distinguish variable roles

A **confounder** is a common cause of exposure and outcome. Appropriate
adjustment can block a backdoor path when the variable is measured adequately.

A **mediator** occurs after exposure and carries part of its effect. Soil
moisture or crop disease may mediate rainfall effects. Adjusting for a mediator
can remove part of a total effect.

A **collider** is caused by two variables. Conditioning on it can create an
association between its causes and open a biased path.

A **proxy** imperfectly represents another concept. Country and year terms
proxy some contextual differences but do not guarantee control of
time-varying confounding.

A **precision variable** predicts the outcome without being required to block
confounding. It may improve precision but is not what makes an estimate causal.

Roles depend on the causal question. Irrigation could be a baseline
confounder, a response to expected rainfall, a mediator, or part of the
intervention. State timing and meaning before adjustment.

---

## Evaluate identification assumptions

### Consistency

If a unit receives exposure \(p\), its observed outcome must correspond to
\(Y(p)\). Equal seasonal totals with different timing, intensity, or spatial
distribution may not be equivalent treatment versions.

### Exchangeability

Conditional exchangeability requires:

\[
Y(p) \perp P \mid C
\]

after adjustment for a sufficient set of pre-exposure common causes \(C\).
This cannot be tested directly. Temperature, irrigation, input use, economic
shocks, crop location, and management remain concerns.

### Positivity

Relevant exposure contrasts must occur within adjustment groups. If a country
never experiences the required precipitation range, the model extrapolates.
For a continuous exposure, inspect distributions and ranges by country and
period.

### No interference

One unit's exposure should not alter another unit's potential outcome under
the treatment definition. Shared water, trade, migration, and regional shocks
can challenge this assumption.

### Measurement and selection

Recorded variables and rows must represent the intended concepts and target
population. Country-area precipitation is not crop-specific exposure, and
national yield conceals subnational variation.

---

## Understand linear regression

For country \(i\) and year \(t\), consider:

\[
Y_{it}=\beta_0+\beta_1P_{it}+\alpha_i+f(t)+\varepsilon_{it}
\]

where \(Y\) is national yield, \(P\) is seasonal precipitation,
\(\alpha_i\) represents country indicators, \(f(t)\) represents time, and
\(\varepsilon\) is the observed deviation from fitted yield.

Ordinary least squares minimizes squared residuals. Fitted values are modeled
conditional means. Residuals are not measurements of causal effects or every
omitted cause.

The intercept is expected yield when numeric variables equal zero and factors
are at reference levels. Centering year and scaling precipitation make the
intercept and slope easier to read without changing fitted values in an
otherwise equivalent linear model.

---

## Interpret coefficients conditionally

When precipitation is expressed per 100 mm, \(\beta_1\) is the modeled
difference in expected yield associated with a 100 mm difference, conditional
on included terms.

It is causal only if:

- the coefficient corresponds to the intended estimand;
- the causal effect is identified by the adjustment strategy;
- functional form and measurement are adequate;
- the estimator and uncertainty procedure are appropriate; and
- selection and interference do not invalidate comparison.

“Holding country and year constant” describes a model comparison. It does not
mean all country characteristics and historical processes have been fixed.

---

## Represent countries and time

Country indicators allow different intercepts and use within-country exposure
variation. They account for stable differences represented by country
membership, not unmeasured country characteristics that change over time.

A common linear trend assumes equal additive annual change across countries.
Country-specific trends relax that assumption but consume more information
and can absorb exposure variation. Flexible time terms may improve fit while
increasing uncertainty or extrapolation.

Repeated country observations can have correlated residuals. Default ordinary
regression intervals may not represent this dependence. With only nine
countries, cluster-based inference also needs caution. Report the limitation
rather than presenting default intervals as definitive.

---

## Check functional form and interactions

A linear precipitation term assumes a constant slope. Agronomic reasoning
suggests possible nonlinearity: rain may help under dry conditions but offer
little benefit or cause damage under wet conditions.

Motivated alternatives include:

- a quadratic precipitation term;
- a spline as an advanced extension;
- precipitation-by-country interactions; and
- alternative time representations.

An interaction means the modeled association differs across another variable
and must be interpreted jointly. Flexibility can address functional form; it
does not solve confounding or exposure ambiguity.

---

## Interpret uncertainty carefully

A confidence interval describes sampling uncertainty under a model and
procedure. It does not quantify uncertainty from:

- unmeasured confounding or a wrong causal graph;
- exposure-version ambiguity;
- selection or interference;
- measurement error; or
- selective model choice.

A p-value evaluates compatibility with a specified null model. It is not the
probability that a causal hypothesis is true. Report estimate, unit, interval,
sample size, model, and identification limitations together.

---

## Use diagnostics for their proper purpose

| Diagnostic | Possible warning | Does not establish |
| --- | --- | --- |
| Residuals versus fitted | Nonlinearity or changing spread | No unmeasured confounding |
| Residual quantiles | Heavy tails or unusual residuals | Causal direction |
| Scale-location plot | Non-constant residual variance | Correct measurement |
| Leverage and Cook's distance | Influential observations | Whether influence is bias |
| Residuals over time | Trend or temporal dependence | Exchangeability |
| Exposure by group | Poor overlap or extrapolation | Consistency |

Investigate warnings rather than deleting observations automatically. An
influential year may be a real drought, policy shift, measurement problem, or
model misspecification.

---

## Compare specifications as sensitivity evidence

Use a planned sequence:

1. unadjusted precipitation association;
2. country indicators;
3. a time representation;
4. country and time together; and
5. motivated nonlinear or interaction terms.

Compare direction, magnitude, uncertainty, residual structure, and exposure
support. Large changes reveal specification dependence. Small changes show
stability across those specifications, not protection against every omitted
variable.

Do not search many models and retain only a significant result. Record the
sequence, rationale, and all planned results.

---

## Bound the causal conclusion

A useful conclusion has five layers:

1. **Observed data:** population, grain, exposure, outcome, and coverage.
2. **Statistical estimate:** association, unit, interval, and model.
3. **Identification assessment:** assumptions supported, doubtful, or unknown.
4. **Causal claim:** causal language only to the justified extent.
5. **Next evidence:** measurements or designs that would strengthen inference.

For this project, a defensible core conclusion is likely:

> The models estimate precipitation-yield associations conditional on selected
> country and time terms. The national observational data do not adequately
> establish a well-defined precipitation intervention or control important
> time-varying common causes, so coefficients are not identified causal effects.

---

## Check your understanding

1. How do an estimand, estimator, and estimate differ?
2. What does a potential outcome represent?
3. Why is a DAG an assumption rather than a fitted result?
4. How do confounders, mediators, and colliders differ?
5. What do consistency, exchangeability, positivity, and no interference require?
6. What does a precipitation coefficient represent in a linear model?
7. Why do country indicators not remove every country-related confounder?
8. How can temporal dependence affect uncertainty?
9. What does a quadratic term address, and what remains unresolved?
10. Why can a narrow confidence interval accompany weak identification?
11. Which questions can residual diagnostics answer?
12. What makes specification comparison useful rather than selective?

---

## Further resources

- Miguel A. Hernán and James M. Robins, [Causal Inference: What If](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/)
- Scott Cunningham, [Causal Inference: The Mixtape](https://mixtape.scunning.com/)
- Brady Neal, [Introduction to Causal Inference](https://www.bradyneal.com/causal-inference-course)
- [DAGitty](https://www.dagitty.net/)
- [R for Data Science (2e): Model basics](https://r4ds.hadley.nz/model-basics.html)
- [R `lm` documentation](https://stat.ethz.ch/R-manual/R-devel/library/stats/html/lm.html)

---

## Continue to Application

Continue with [Conduct a causal analysis of maize
yield](09_03_explanatory_analysis_application.md). You will define an estimand,
draw a causal diagram, assess identification, compare regression
specifications, inspect diagnostics, and write a bounded conclusion.
