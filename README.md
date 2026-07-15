# Bayesian Hierarchical Modeling of MLB Pitch Velocity

Final project for an undergraduate Bayesian Statistics class ("365"), dated 2025-05-02.

## Question

How much of a pitch's release speed comes down to in-game situational factors (count, inning, outs, baserunners, score), versus the pitcher's own ability and the pitch type thrown?

## Data

Statcast pitch-level data, July 2024, from eight pitchers chosen to span the league's skill range: Tarik Skubal, Paul Skenes, Patrick Corbin, Bryce Elder, Kyle Gibson, Logan Gilbert, Jordan Montgomery, and Zac Gallen. Response variable is release speed (mph). Predictors: pitch type (nine types, changeup as baseline), balls, strikes, outs, inning, runner-on-base indicators, batting-team score, fielding-team score (both standardized), and pitcher identity for random effects. Two screwballs were dropped as noise.

The raw CSV (`savant_data (6).csv`) and the JAGS model definition files (`fixed_effects.txt`, `rand_effects.txt`) aren't included here.

## Method

Four Bayesian linear regression models (normal likelihood), fit in JAGS via `rjags` with three chains, 1,000 burn-in and 5,000 sampled iterations, varying in whether they include pitch type and/or pitcher-specific random intercepts:

| Model | Pitch Type | Random Pitcher Effects | DIC |
|---|---|---|---|
| 1 | No | No | 30,594 |
| 2 | Yes | No | 28,938 |
| 3 | No | Yes | 19,970 |
| 4 | Yes | Yes | **13,393** |

Fixed-effect coefficients used weakly informative N(0, 0.01) priors (JAGS precision parameterization); residual and random-effect precisions used Gamma(0.001, 0.001), with the random-effect SD on a dunif(0, 40) prior. Convergence checked via Gelman-Rubin (all fixed effects under the 1.1 threshold, most under 1.05), effective sample size, and trace plots.

## Results

Model 4 (pitch type plus pitcher random intercepts) won decisively, DIC dropping from 30,594 to 13,393, better than a 50% reduction, and was used for all inference. That result intuitively makes sense: it matches how coaches and scouts actually think about velocity, a pitch type's own typical speed adjusted for the fact that not everyone throws like Paul Skenes.

Significant predictors (95% credible interval excludes zero): most pitch types relative to the changeup baseline (cutters, four-seamers, splitters, sinkers, curveballs/knuckle-curves, sweepers all showed strong, distinct effects), more strikes in the count implying higher velocity, more outs implying higher velocity, later innings implying lower velocity likely due to fatigue, and a small positive effect from a fielding-team lead.

Not significant: sliders, which didn't separate from the changeup baseline, all three baserunner indicators, and batting-team score. Velocity doesn't move much with runners on or when trailing at the plate.

Comparing Model 3 against Model 4 for a fixed scenario (four-seam fastball, 1-1 count, 2 outs, 3rd inning), Model 4 predicted 4 to 6 mph higher release speeds per pitcher than Model 3, confirming that leaving out pitch type systematically understates fastball velocity, and that Model 3's narrower credible intervals were an artifact of missing that pitch-specific variation rather than genuine extra precision.

## Contents

- `pitch-velocity-bayesian-hierarchical-model.Rmd` — full analysis: data prep, JAGS model specification, convergence diagnostics, and inference.

## Author

Jack Huffard
