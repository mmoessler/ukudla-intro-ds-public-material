# 9.2) Understand explanatory-modeling concepts

---

- Source: [09_02_explanatory_analysis_concepts.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/09_02_explanatory_analysis_concepts.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/09_02_explanatory_analysis_concepts.md)
- Feedback: [Topic 09: Explanatory Analysis](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/10)
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
- [Represent the study design and dependence](#represent-the-study-design-and-dependence)
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
[the maize-yield worked example](09_03_explanatory_analysis_application.md).

Use one organizing principle:

> A statistical model receives causal meaning from the question, design, and
> identification assumptions—not from the model family or coefficient name.

---

## Separate question, estimand, identification, and estimation

These stages answer different questions:

| Stage | Question | General example |
| --- | --- | --- |
| Causal question | What change and outcome should be understood? | What would happen under treatment rather than comparison condition? |
| Estimand | Which causal quantity summarizes the question? | Average difference in potential outcomes under two conditions |
| Identification | Under which assumptions can observed data express it? | Random assignment or conditional exchangeability and overlap |
| Estimator | Which procedure calculates the identified quantity? | Design-based comparison or regression adjustment under a stated model |
| Estimate | What value resulted in this sample? | Estimated outcome contrast with uncertainty |

An estimator can be computed when identification fails. It then estimates an
observed-data association, not the intended causal estimand.

---

## Use potential outcomes to define causal effects

Let \(Y_i(a)\) represent the outcome unit \(i\) would have under treatment or
exposure condition \(a\). For two specified conditions, the unit-level effect
is:

\[
Y_i(a_1)-Y_i(a_0)
\]

The average causal effect over a target population is:

\[
E\left[Y(a_1)-Y(a_0)\right]
\]

The same unit cannot reveal both outcomes at the same time. Causal inference
uses other observed units or times as comparisons under assumptions.

For a continuous exposure, one might target a specified increment or a
dose-response function \(E[Y(a)]\). A constant linear slope assumes the same
marginal contrast across the supported range. That assumption needs domain and
design justification.

---

## Make causal structure explicit

A **directed acyclic graph** (DAG) represents assumed causal relationships:

- nodes represent variables or concepts;
- arrows represent direct causal influence at the chosen abstraction;
- paths represent possible routes of association; and
- acyclic means the represented time order contains no feedback loop.

A generic DAG could contain:

~~~text
pre-treatment causes C ─────► treatment or exposure A ─────► outcome Y
          │                                                    ▲
          └────────────────────────────────────────────────────┘

treatment A ─────► mediator M ─────► outcome Y

treatment A ─────► selection S ◄───── outcome Y
~~~

The diagram is not discovered by selecting significant correlations. It is a
claim based on domain knowledge, temporality, measurement, and the research
question. Record plausible alternatives.

A broad node such as “site or background conditions” hides many mechanisms.
Drawing it does not mean one indicator or adjustment term measures every
relevant cause.

---

## Distinguish variable roles

A **confounder** is a common cause of exposure and outcome. Appropriate
adjustment can block a backdoor path when the variable is measured adequately.

A **mediator** occurs after exposure and carries part of its effect. A
physiological response, intermediate soil condition, or changed practice may
mediate an intervention effect. Adjusting for a mediator can remove part of a
total effect.

A **collider** is caused by two variables. Conditioning on it can create an
association between its causes and open a biased path.

A **proxy** imperfectly represents another concept. Site, batch, group, or time
terms may represent some contextual differences but do not guarantee control
of unmeasured confounding.

A **precision variable** predicts the outcome without being required to block
confounding. It may improve precision but is not what makes an estimate causal.

Roles depend on the causal question. A management practice, laboratory
condition, or physiological measurement could be a baseline confounder, a
response to treatment, a mediator, or part of the intervention. State timing
and meaning before adjustment.

---

## Evaluate identification assumptions

### Consistency

If a unit receives condition \(a\), its observed outcome must correspond to
\(Y(a)\). Treatments with different dose, timing, delivery, adherence, or
co-interventions may not be equivalent versions.

### Exchangeability

Conditional exchangeability requires:

\[
Y(a) \perp A \mid C
\]

after adjustment for a sufficient set of pre-exposure common causes \(C\).
Random assignment can support this condition in an experiment; observational
studies require substantive justification and measurement of relevant common
causes. The condition cannot be established from the fitted model alone.

### Positivity

Relevant treatment or exposure contrasts must occur within adjustment groups.
Inspect assignment cells or exposure support across blocks, sites, batches,
sampling strata, and other important groups. Unsupported contrasts require
extrapolation.

### No interference

One unit's treatment should not alter another unit's potential outcome under
the treatment definition. Contamination between laboratory samples, spillover
between plots or farms, and shared environmental or social systems can
challenge this assumption.

### Measurement and selection

Recorded variables and rows must represent the intended concepts and target
population. Instrument readings, assigned treatments, observed practices, and
provider aggregates can each differ from the construct named in the causal
question.

---

## Understand linear regression

For target unit \(i\), consider:

\[
Y_i=\beta_0+\beta_1A_i+\boldsymbol{\gamma}^{\mathsf T}\mathbf{C}_i+\varepsilon_i
\]

where \(Y\) is the outcome, \(A\) is treatment or exposure,
\(\mathbf{C}\) contains justified pre-exposure covariates, and
\(\varepsilon\) is the observed deviation from the fitted outcome.

Ordinary least squares minimizes squared residuals. Fitted values are modeled
conditional means. Residuals are not measurements of causal effects or every
omitted cause.

The intercept is the expected outcome when numeric variables equal zero and
factors are at reference levels. Centering or scaling variables can make the
intercept and slope easier to read without changing fitted values in an
otherwise equivalent linear model.

---

## Interpret coefficients conditionally

For a binary treatment, \(\beta_1\) is the modeled conditional mean difference;
for a continuous exposure it is the modeled difference per stated unit,
conditional on included terms.

It is causal only if:

- the coefficient corresponds to the intended estimand;
- the causal effect is identified by the adjustment strategy;
- functional form and measurement are adequate;
- the estimator and uncertainty procedure are appropriate; and
- selection and interference do not invalidate comparison.

“Holding covariates constant” describes a model comparison. It does not mean
that all common causes have been measured or physically controlled.

---

## Represent the study design and dependence

An analysis should preserve randomization, blocking, clustering, repeated
measurements, batches, sites, sampling strata, and time when they define the
comparison or uncertainty. Fixed or random group terms answer particular
questions; they are not interchangeable corrections for every dependency.

Repeated observations and grouped units can have correlated residuals, so
default ordinary-regression intervals may not represent uncertainty. The
number of independent experimental or sampling units—not merely the row
count—constrains what can be learned. State the design and justify the
uncertainty procedure.

---

## Check functional form and interactions

A linear exposure term assumes a constant slope. Domain reasoning may instead
suggest thresholds, saturation, diminishing returns, or harm at high doses.

Motivated alternatives include:

- a quadratic exposure term;
- a spline as an advanced extension;
- treatment-by-site or exposure-by-group interactions; and
- alternative representations of batch, block, space, or time.

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
influential observation may be a real response, protocol deviation,
measurement problem, unusual context, or model misspecification.

---

## Compare specifications as sensitivity evidence

Use a planned sequence:

1. unadjusted treatment or exposure association;
2. terms required by assignment or sampling design;
3. justified pre-exposure common causes;
4. the planned primary specification; and
5. motivated nonlinear, interaction, or sensitivity specifications.

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

A defensible conclusion can therefore state that a randomized comparison
supports a specified treatment effect, that an observational analysis supports
an effect only under explicit assumptions, or that the available data identify
only an adjusted association. Making the boundary visible is a result rather
than a failure.

---

## Check your understanding

1. How do an estimand, estimator, and estimate differ?
2. What does a potential outcome represent?
3. Why is a DAG an assumption rather than a fitted result?
4. How do confounders, mediators, and colliders differ?
5. What do consistency, exchangeability, positivity, and no interference require?
6. What does an exposure coefficient represent in a linear model?
7. Why do group indicators not remove every group-related confounder?
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

Continue with the [maize-yield worked
example](09_03_explanatory_analysis_application.md). You will define an estimand,
draw a causal diagram, assess identification, compare regression
specifications, inspect diagnostics, and write a bounded conclusion.
