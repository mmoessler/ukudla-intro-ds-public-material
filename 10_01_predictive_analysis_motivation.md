# 10.1) Why predictive modeling requires unseen-data evaluation

---

- Last Update: 2026-09-03
- Source: [10_01_predictive_analysis_motivation.md](/learning-modules/intro-ds-module/10_01_predictive_analysis_motivation.md)
- Estimated reading time: 20 minutes
- Estimated activity time: 10 minutes

---

## Outline

- [Outline](#outline)
- [Learning objectives](#learning-objectives)
- [Place in the session](#place-in-the-session)
- [Prediction concerns observations not used for fitting](#prediction-concerns-observations-not-used-for-fitting)
- [Explanation and prediction are different objectives](#explanation-and-prediction-are-different-objectives)
- [Intended use determines meaningful evaluation](#intended-use-determines-meaningful-evaluation)
- [Dependence and transfer make prediction difficult](#dependence-and-transfer-make-prediction-difficult)
- [Simple baselines are scientifically useful](#simple-baselines-are-scientifically-useful)
- [What can go wrong](#what-can-go-wrong)
- [Worked example: the maize-yield project](#worked-example-the-maize-yield-project)
- [Initial activity](#initial-activity)
- [Check your understanding](#check-your-understanding)
- [Further resources](#further-resources)
- [Continue to Concepts](#continue-to-concepts)

---

## Learning objectives

After completing this page, you should be able to:

- distinguish a predictive objective from a descriptive or causal objective;
- explain why training-data performance does not establish predictive usefulness;
- connect prediction time and intended use to the information available to a model;
- justify evaluating later observations instead of a random sample of country-years; and
- recognize leakage, distribution shift, and an underspecified prediction task.

---

## Place in the session

This is the **Motivation** part of the Predictive Modeling session:

~~~text
Motivation  →  Concepts  →  Application
    ↑
 this page
~~~

The Explanatory Modeling session asked what a modeled relationship could mean
under explicit causal assumptions. Predictive modeling asks a different
question: how accurately can a procedure predict outcomes that were not used
to construct it?

[Understand predictive-modeling concepts](10_02_predictive_analysis_concepts.md)
defines a prediction contract, data splits, leakage, baselines, and evaluation
metrics across study designs. The [application](10_03_predictive_analysis_application.md)
demonstrates those concepts with the maize project.

---

## Prediction concerns observations not used for fitting

A fitted model can describe its training observations very closely and still
fail on new observations: it may have learned stable structure, or instead
noise, unusual cases, or information unavailable when a prediction is needed.

Predictive performance therefore concerns **generalization** — performance on
relevant observations outside the data used for fitting and model selection.
Learners must define the intended use, protect an evaluation set, fit every
learned operation using training data only, generate predictions without
seeing the answers, and then quantify the errors.

The useful question is not how closely the fitted line follows its estimation
data, but how well the complete procedure predicts observations representing
its intended use.

---

## Explanation and prediction are different objectives

Explanatory and predictive models can use the same mathematical family, such
as linear regression, while serving different purposes. An explanatory
analysis needs a defensible intervention, causal structure, and an estimator
that matches the estimand — for example, whether a precipitation contrast has
a causal effect on maize yield. A predictive analysis needs a precise
prediction task and evaluation on representative unseen data — for example,
how accurately later national yields can be predicted from information
available at a stated time.

Neither objective is automatically stronger: a variable can improve
predictions without causing the outcome, a causal variable can add little
predictive accuracy, and predictive accuracy cannot repair an unidentified
causal analysis. The objective must be chosen before selecting models and
performance measures.

---

## Intended use determines meaningful evaluation

“Predict maize yield” is incomplete. A prediction task must say what outcome
and unit are predicted, which countries and years form the target population,
when the prediction is made and how far ahead it reaches, which information
exists at that time, how data are separated for development and evaluation,
and which errors matter for the intended decision.

For example, realized annual precipitation cannot be used for a forecast made
before the rainy season because those observations do not yet exist, though it
may be valid for a post-season estimate — a different task even with the same
outcome and model.

Evaluation must reproduce the intended use as closely as the teaching data
allow: if use concerns later years, randomly mixing early and late
country-years between training and test sets can make performance appear more
reliable than it is.

---

## Dependence and transfer make prediction difficult

The intended deployment boundary differs across projects. Laboratory models
may need to transfer to new biological samples, instruments, or batches. Field
models may need to transfer to new plots, sites, seasons, or farms. Secondary
data may require prediction for later periods or previously unseen entities.

Related rows cannot be treated automatically as independent: aliquots can
share a biological sample, plots can share a block or site, and repeated visits
can share a unit. An evaluation split must keep these relationships from
leaking information across development and test data.

The maize-yield example emphasizes temporal transfer:

The maize panel combines repeated annual observations for a small set of
countries, where neighboring years are related and technology, management,
markets, climate, and reporting systems can alter both the level and
variability of yield over time.

Consequently, the future may not follow the same distribution as the past — a
possibility called **distribution shift**. The Descriptive Data Analysis
session examined trends, period differences, changing variance, and temporal
dependence because each affects later-period prediction.

A temporal split does not eliminate shift; it exposes the model to one
realistic form of it by learning from earlier observations and evaluating on
later ones. Good performance for 2018–2022 still does not guarantee good
performance after 2022, in new countries, or under unprecedented conditions.

---

## Simple baselines are scientifically useful

A complex model is not useful merely because it can produce predictions — it
should improve on simple, credible rules under the same evaluation design.
For maize yield: does it beat one historical average? Does a common time
trend help? Does accounting for persistent country differences help further?
Does an added predictor improve held-out errors enough to justify its cost?

Baselines reveal how much performance comes from simple regularities. If a
complex model cannot beat them, complexity has not earned its cost; if it
does, the size and stability of the improvement still matter.

---

## What can go wrong

Common failures include:

- reporting training fit as predictive performance;
- choosing or repeatedly tuning models against the test period after
  inspecting its outcomes;
- preprocessing all years before splitting the data, or using variables
  measured after the stated prediction time;
- allowing duplicate or related observations to cross split boundaries;
- selecting a model without comparing a meaningful baseline, or reporting
  only one aggregate metric that hides country-specific failures;
- interpreting predictive associations causally; and
- deploying beyond the countries, years, and conditions represented by the
  evaluation.

These are workflow and design problems, not ones a more sophisticated
algorithm automatically solves.

---

## Worked example: the maize-yield project

The example project defines a transparent teaching benchmark:

- predict annual national log maize yield;
- use country identity and calendar year as the available information;
- train on 1990–2017 observations;
- hold out 2018–2022 observations;
- compare a historical mean, a common linear trend, and a country-plus-time
  model; and
- evaluate row-level errors with mean absolute error (MAE) and root mean
  squared error (RMSE).

This is not an operational crop forecast: the split contains only five test
years per country, the predictors are deliberately limited, and calendar year
is a trend representation rather than a mechanism, teaching an honest
predictive workflow and the boundaries of its evidence.

Precipitation can support a separate extension only once its availability at
prediction time is specified — realized seasonal precipitation supports a
nowcasting or retrospective task, not necessarily an early-season forecast.

---

## Initial activity

Write a short prediction contract for one of these uses:

1. forecast annual maize yield before the growing season;
2. update the prediction during the growing season; or
3. estimate yield after the season but before official yield data are published.

State the target, prediction time, horizon, available information, target
countries, and one important error. Then answer:

- Would realized growing-season precipitation be available?
- Would a random train/test split represent the use?
- Which simple rule should the model beat?

---

## Check your understanding

1. Why can a regression with excellent training fit predict later years poorly?
2. Why does prediction time determine whether precipitation is a valid predictor?
3. What does a temporal holdout test that a random holdout may not?
4. Why does predictive performance not establish a causal effect?
5. What makes the current maize exercise a teaching benchmark rather than an operational forecast?

---

## Further resources

- [An Introduction to Statistical Learning](https://www.statlearning.com/) introduces prediction, resampling, and model assessment.
- [Tidy Modeling with R](https://www.tmwr.org/) presents reproducible model development and evaluation in R.
- [The Turing Way: Reproducible Research](https://book.the-turing-way.org/reproducible-research/reproducible-research/) explains auditable, reusable computational practice.
- [scikit-learn: Common pitfalls and recommended practices](https://scikit-learn.org/stable/common_pitfalls.html) gives examples of preprocessing leakage that apply outside Python.
- Shmueli, G. (2010). “To Explain or to Predict?” *Statistical Science*, 25(3), 289–310, clarifies why explanatory and predictive modeling need different evaluation criteria.

---

## Continue to Concepts

Continue with [Understand predictive-modeling
concepts](10_02_predictive_analysis_concepts.md) to define the task, protect unseen
data, and interpret predictive errors.
