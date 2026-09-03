# 10.2) Understand predictive-modeling concepts

---

- Source: [10_02_predictive_analysis_concepts.md](https://github.com/mmoessler/ukudla-intro-ds-public-material/blob/main/10_02_predictive_analysis_concepts.md)
- History: [Commit History](https://github.com/mmoessler/ukudla-intro-ds-public-material/commits/main/10_02_predictive_analysis_concepts.md)
- Feedback: [Topic 10: Predictive Analysis](https://github.com/mmoessler/ukudla-intro-ds-public-material/discussions/11)
- Estimated reading time: 60 minutes
- Estimated activity time: 30 minutes

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Begin with a prediction contract](#begin-with-a-prediction-contract)
- [Separate training, validation, and testing](#separate-training-validation-and-testing)
- [Choose a split that represents use](#choose-a-split-that-represents-use)
- [Prevent leakage](#prevent-leakage)
- [Evaluate the complete workflow](#evaluate-the-complete-workflow)
- [Use baselines to define useful performance](#use-baselines-to-define-useful-performance)
- [Understand prediction errors, MAE, and RMSE](#understand-prediction-errors-mae-and-rmse)
- [Evaluate errors beyond one average](#evaluate-errors-beyond-one-average)
- [Recognize overfitting and distribution shift](#recognize-overfitting-and-distribution-shift)
- [Handle transformed outcomes carefully](#handle-transformed-outcomes-carefully)
- [Define reproducible prediction artifacts](#define-reproducible-prediction-artifacts)
- [Bound interpretation and use](#bound-interpretation-and-use)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Application](#continue-to-application)

---

## Learning objectives

After completing this page, you should be able to:

- specify a prediction contract with a target, information set, horizon, use, and evaluation design;
- distinguish training, validation, and test data;
- select a temporal or grouped split that represents the deployment problem;
- identify target, preprocessing, feature, and test-set leakage;
- compare candidate models with transparent baselines;
- calculate and interpret prediction errors, MAE, and RMSE;
- inspect performance across observations, countries, and periods;
- explain overfitting, temporal dependence, and distribution shift;
- handle log-scale predictions deliberately; and
- define the artifacts needed to reproduce a predictive evaluation.

---

## Place in the session

This is the **Concepts** part of the Predictive Modeling session:

~~~text
Motivation  →  Concepts  →  Application
                ↑
             this page
~~~

[Why predictive modeling requires unseen-data
evaluation](10_01_predictive_analysis_motivation.md) introduced the need for an
intended use and representative holdout. This page supplies the concepts used
in [the maize-yield worked example](10_03_predictive_analysis_application.md).

Use one organizing principle:

> Predictive performance is a property of a complete procedure, evaluated on
> relevant unseen observations—not a property of a fitted equation alone.

---

## Begin with a prediction contract

A **prediction contract** is a documented agreement about what will be
predicted, when, from which information, for whom, and how success will be
judged. It prevents the task from changing after results are visible.

| Component | Question | General specification |
| --- | --- | --- |
| Target | What value and scale are predicted? | A named outcome on a defined scale |
| Unit | What receives one prediction? | A sample, plot, farm, visit, entity, or period |
| Target population | Where should predictions apply? | The future deployment population |
| Prediction time | When is the prediction issued? | Before the outcome and unavailable predictors are observed |
| Horizon | How far ahead is the outcome? | A specified interval or transfer setting |
| Information set | What is available then? | Only measurements available at prediction time |
| Evaluation | Which observations remain unseen? | Units or periods representing intended use |
| Loss and metrics | Which errors matter? | Loss tied to the decision and outcome scale |
| Intended use | Which decision will use it? | A stated supported use and exclusions |

The limitation in the horizon is important. Calling this workflow a forecast
would imply an operational issue date and information cutoff that the exercise
does not fully define. “Held-out prediction benchmark” is more accurate.

The **information set** contains only predictors legitimately known at the
prediction time. A value can exist in a historical table but be unavailable
when a real prediction would be issued. Realized growing-season precipitation,
for example, cannot support a pre-season forecast. It can support a separately
defined post-season estimate or nowcast.

---

## Separate training, validation, and testing

The three data roles answer different questions:

| Partition | Primary role | Permitted use |
| --- | --- | --- |
| Training | Learn parameters and transformations | Fit means, coefficients, imputers, encodings, or other learned operations |
| Validation | Compare candidates and settings | Select models or hyperparameters without using the final test set |
| Test | Estimate final generalization | Evaluate the fixed workflow once |

With a small dataset, validation may use resampling within the training
period rather than a permanent third partition; for temporal data,
rolling-origin or forward-chaining resampling usually represents later
prediction better than ordinary random cross-validation.

A small benchmark can compare predefined candidates without extensive tuning,
but its test observations must still remain excluded from fitting and
candidate invention. Repeatedly changing the workflow after viewing test
errors turns the test data into validation data and removes the independent
final assessment.

---

## Choose a split that represents use

A split is part of the scientific design. Common strategies include:

- **random split** when observations are exchangeable and use resembles a
  random observation from the same population;
- **group split** when all observations from a person, site, farm, or country
  must remain together;
- **temporal split** when training on earlier observations and predicting later
  ones represents use;
- **spatial split** when geographic transfer is the objective; and
- **rolling-origin evaluation** when several successive past-to-future tests
  are needed.

Choose the boundary that matches use: holding out aliquots does not evaluate
new biological samples, holding out plots may not evaluate a new site, and a
temporal split for known entities does not evaluate transfer to new entities.

Do not select the split because it produces favorable metrics; record its
boundary, rationale, and exclusions before inspecting test outcomes.

---

## Prevent leakage

**Data leakage** occurs when model development uses information unavailable
under the prediction contract. It creates unrealistically good results.

Common routes include:

- **target leakage**: a predictor directly or indirectly contains the outcome;
- **future leakage**: later information is used to predict an earlier outcome;
- **preprocessing leakage**: means, scales, imputation values, categories, or
  selected variables are learned from all data before splitting;
- **group leakage**: related or duplicated observations appear on both sides;
- **test-set leakage**: test outcomes influence features, candidates, or tuning;
  and
- **publication leakage**: decisions are revised after test performance is
  known but reported as if they were prespecified.

Prevent leakage by splitting before learned preprocessing, fitting the
complete recipe on training data only, preserving group and time boundaries,
and logging every interaction with the test set.

Deterministic operations defined independently of observed values — such as a
documented unit conversion — can apply to both partitions; operations that
estimate something from a distribution must learn it from training only.

---

## Evaluate the complete workflow

A predictive workflow includes more than `lm()` or another model call:

~~~text
prediction contract
        ↓
split definition
        ↓
training-fitted preparation
        ↓
candidate model fitting
        ↓
prediction on unseen rows
        ↓
error calculation and subgroup checks
        ↓
artifacts, limitations, and report
~~~

Performance belongs to this whole sequence. Changing the split, imputation,
feature definition, or prediction scale changes the evaluated procedure even
when the model formula remains unchanged.

---

## Use baselines to define useful performance

A **baseline** is a simple prediction rule that an elaborate candidate should
improve upon. It provides context that a metric alone cannot.

The maize workflow uses:

- a **historical mean**, which predicts one training-period mean log yield;
- a **common linear trend**, which extrapolates a trend across countries; and
- a **country-plus-time model**, which represents persistent country
  differences and a common temporal trend.

Other defensible baselines include the last observed value or a per-country
training mean — a country mean asks whether identity alone is sufficient, and
a last-value rule asks whether the most recent observation is strongly
predictive.

Candidates must use the same split, target rows, outcome scale, and metric. A
more complex model earns its cost only through material and reasonably stable
unseen-data improvement, not lower training error.

---

## Understand prediction errors, MAE, and RMSE

For observation \(i\), define prediction error as:

\[
e_i = y_i - \hat{y}_i,
\]

where \(y_i\) is observed and \(\hat{y}_i\) is predicted. Positive errors mean
underprediction and negative errors mean overprediction under this convention.

Mean absolute error is:

\[
\mathrm{MAE} = \frac{1}{n}\sum_{i=1}^{n}|y_i-\hat{y}_i|.
\]

Root mean squared error is:

\[
\mathrm{RMSE} =
\sqrt{\frac{1}{n}\sum_{i=1}^{n}(y_i-\hat{y}_i)^2}.
\]

Both are non-negative; lower is better on the same test set. MAE weights
absolute errors linearly. RMSE squares errors and emphasizes large misses.
Neither is universally best. Choose loss to reflect intended use and report
complementary evidence.

Signed mean error can reveal systematic over- or underprediction, but positive
and negative values can cancel. In this project all core metrics are calculated
on `log_yield`, so they are not errors in tonnes per hectare.

---

## Evaluate errors beyond one average

One aggregate metric can hide important failure modes. Inspect:

- row-level observations, predictions, and errors;
- errors by treatment, batch, site, group, or period;
- error distributions and extreme misses;
- systematic over- or underprediction;
- performance for high and low target values; and
- whether candidate rankings change across subgroups.

Subgroup results with few independent test observations are diagnostic signals,
not precise performance claims. Always report their denominators.
Observed-versus-predicted plots can expose bias and missed structure that MAE
and RMSE compress into one number.

---

## Recognize overfitting and distribution shift

**Overfitting** occurs when a workflow adapts to training details that do not
generalize: flexible models can reduce training error while increasing unseen
error, and simpler models can **underfit** by omitting stable predictive
structure. Validation evidence, not test-set optimization, should balance
these risks.

Related observations may be dependent—technical replicates share a sample,
plots share a block or site, and repeated visits share a unit. A random row
split can place close relatives on both sides and overstate performance.

**Distribution shift** means inputs, outcomes, or their relationship change
between development and use. New batches, instruments, sites, seasons,
populations, practices, and measurement protocols can all create shift;
compare periods descriptively, but do not redesign the final evaluation
silently after inspecting test outcomes.

Stability cannot be assumed from one finite dataset. The design should
anticipate change, and the conclusion should stay bounded to the evaluated
units, settings, and periods.

---

## Handle transformed outcomes carefully

The project models `log_yield`. This can stabilize variance and represent
multiplicative differences additively, but it changes interpretation.

Exponentiating a predicted mean log outcome does not generally equal the mean
outcome on the original scale. A naive back-transformation can be biased due to
the nonlinear exponential function. For the core exercise, compare candidates
consistently on the log scale and state that scope.

If an intended use requires tonnes per hectare, define the desired quantity,
justify a back-transformation method, and evaluate predictions on that scale as
an extension. Do not combine log-scale and original-scale metrics in one model
ranking.

---

## Define reproducible prediction artifacts

An auditable evaluation should preserve:

- the prediction contract and intended exclusions;
- a deterministic split rule or split manifest;
- versions and checksums for input data;
- feature and preprocessing definitions;
- fitted objects or enough code to recreate them;
- one row-level table containing keys, observations, and predictions;
- overall and subgroup metric tables;
- diagnostic figures;
- software environment information; and
- a report stating limitations and intended use.

Artifacts should be generated by scripts rather than edited manually. Stable
keys must connect predictions to source rows, and the workflow should fail when
required inputs, years, or columns are absent.

---

## Bound interpretation and use

The maize evaluation supports one narrow statement: how three predefined
procedures performed for known project countries in 2018–2022 after being
fitted on 1990–2017 observations.

It does not establish:

- performance in countries absent from training;
- performance after 2022;
- operational pre-season forecast accuracy;
- robustness to unprecedented conditions;
- a causal effect of year, country, or precipitation; or
- suitability for high-stakes resource allocation.

Model documentation should say where the workflow was evaluated, where it can
fail, which evidence supports it, and when it needs reevaluation.

---

## Check your understanding

1. Which elements turn “predict maize yield” into a prediction contract?
2. Why is 2018–2022 excluded from both fitting and model invention?
3. Which task would a leave-country-out split evaluate?
4. Give two leakage routes involving seasonal precipitation.
5. Why must preprocessing estimates be learned from training data only?
6. How do MAE and RMSE weight large errors differently?
7. Why can aggregate performance hide an unsuitable model?
8. What exactly does the current temporal test support?
9. Why is exponentiating a mean log prediction not automatically an unbiased mean-yield prediction?
10. Which artifacts allow another analyst to reproduce the evaluation?

---

## Further resources

- [An Introduction to Statistical Learning](https://www.statlearning.com/) covers the bias–variance trade-off, resampling, regression, and model assessment.
- [Tidy Modeling with R](https://www.tmwr.org/) explains data splitting, resampling, recipes, tuning, and evaluation using tidymodels.
- [Forecasting: Principles and Practice](https://otexts.com/fpp3/) provides an open treatment of time-aware evaluation, residuals, accuracy, and forecasting workflows.
- [scikit-learn: Model selection and evaluation](https://scikit-learn.org/stable/model_selection.html) is a useful language-independent reference despite its Python examples.
- [The Turing Way: Reproducible Research](https://book.the-turing-way.org/reproducible-research/reproducible-research/) connects reproducibility, documentation, and transparent computational workflows.
- Kuhn, M. and Johnson, K. (2013). *Applied Predictive Modeling*. Springer, provides a practical treatment of preprocessing, resampling, and evaluation.

---

## Continue to Application

Continue with [Evaluate predictive models for maize
yield](10_03_predictive_analysis_application.md) to inspect the contract, run the
benchmark workflow, and evaluate its evidence.
