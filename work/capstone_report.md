# Capstone Report — <your lane>

- **Author:** Abdelrhman Moubarak
- **Lane:** 
- **Repo:**
- **Date:**

> Copy this file to `work/capstone_report.md` and fill it in as you build. The eight
> sections mirror the Pass / Needs-Work rubric axes, so nothing here is optional.

## 1. Problem framing

This project helps decide which content pages should be refreshed first. The team has limited time, so they need to know which pages are likely to lose search clicks soon, before it happens. The unit of analysis is one content page for one client, looked at over one month. The output is a ranked list of pages, from most likely to decline to least likely, and an editor uses it by working down the list and refreshing the top pages first.

Getting this wrong has a real cost. If the model says a page needs refreshing but it did not actually decline, the team wastes time on it instead of a page that really needed help. If the model misses a page that is truly declining, that page keeps losing clicks with nobody noticing.

A model can help here because many things affect a page's search performance at the same time, like its age, its type, how visible it already is, and its position in search results. A simple fixed rule cannot easily combine all of these the way a model can. That is what this project tests: does a model do better than FlyRank's existing rule?


## 2. Data safety

We used two tables. `fact_content_daily_performance` gives daily search and site metrics for each content page, and we only used March 2026 data, split into two halves. The first half, March 1 to 15, is where we built our features, and the second half, March 16 to 31, is where we checked if clicks went up or down to build our label. `dim_content` gives static information about each page, like its creation date, content type, word count, backlinks, and competition score. We left out a few columns on purpose. We did not use `total_clicks_h1` as a feature, because a page with zero clicks can never be marked as declining under our label, no matter what else happens to it, so keeping it would let the model just learn the label's own rule instead of something real. We also left out `avg_engagement_rate_h1`, since it was missing for 84% of rows, and `search_volume`, `backlinks`, `competition`, and `word_count` were tested as possible features but not kept, which we explain in Section 4. The features we kept are `content_age_days`, `content_type`, `total_impressions_h1`, `has_impressions`, `avg_position_h1`, and one combined feature multiplying age by impressions, `age_x_impressions`.

We checked for leakage risks before trusting any of this. Our label only uses clicks from the second half of March, which comes strictly after the feature window, so there is no direct leakage there. We also checked if any pages were manually updated or optimized during our study window, since that could quietly explain a change in clicks that has nothing to do with our model, and found only 0.29% of pages updated during the first half, 0.02% during the second half, and the earliest optimization event in the whole dataset happening in April, after our study window ends. `client_hash_id` and `content_hash_id` are only used to group pages by **client** for our train and test split, and to join tables together, never as model features, and no client names, domains, URLs, or raw data exports appear anywhere in our files.


## 3. Baseline

Before building any model, we needed a simple rule to compare against, so we built one ourselves. We looked at the columns available to us and picked two: age bucket and content type, since these showed the clearest signal on their own out of everything we checked. The rule sorts every page into one of three tiers based on just these two things: **no action**, **monitor**, or **refresh**, giving us a simple starting point that any editor could understand and use without any model at all.

This rule is a fair comparison because it only uses two simple signals, so beating it means our model found real value in combining more information together. On our test split, the refresh tier, the tier meant to flag pages needing attention, gets a precision of `0.153`, meaning about 15% of the pages it flags actually declined. Picking pages at random on the same split would give a precision of `0.106`, so the rule does a little better than random, about 1.4 times better. We also tried a second, "forced" version of the rule, sorting refresh tier pages by score and only taking the top 50, the same number our model ranks. This gives a precision of `0.26`, but this number is not really fair to trust, because **11,176** pages tie for the exact same top score, so taking "the top 50" out of 11,000+ tied pages is really just picking 50 pages at random from that group. We only show this number for comparison, not as a real result.


## 4. Model / analysis

Your method and why it fits the lane. The exact feature list (and what you left out on
purpose). The target or proxy definition, in one sentence.

## 5. Evaluation

Your split (grouped by client? time-aware?) and why. Metrics, model vs baseline **on the same
split**. What the errors look like — a short error analysis beats a big metric table.

## 6. Interpretation

What the model/clusters actually found. Feature importances or cluster profiles in plain
words. Surprises and negative results — a well-understood "no effect" is a valid result.

## 7. Recommendation

The ranked actions or decisions your output supports, and how a FlyRank editor would use them
tomorrow. State your confidence and the limits explicitly.

## 8. Reproducibility

The exact commands to re-run everything from a fresh clone, your random seeds, and your
environment (`pip freeze` highlights or `requirements.txt` deltas).

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
