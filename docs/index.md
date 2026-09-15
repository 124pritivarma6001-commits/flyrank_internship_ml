# Capstone Report — CTR / Engagement Opportunity Scoring

- **Author:** Priti Varma
- **Lane:** CTR / Engagement Opportunity Scoring
- **Repo:** https://github.com/124pritivarma6001-commits/flyrank_internship_ml
- **Date:** 14 September 2026

## 0. Abstract

This capstone asks how existing content with high search visibility but relatively low click-through rate (CTR) and good average search position can be identified and prioritized for content refresh. The analysis uses the FlyRank internship warehouse, aggregated to 309,234 client-content records using impressions, clicks, CTR, and average search position. A transparent rule-based baseline was compared with a Decision Tree model using impressions, CTR, and average position, with validation including a client-grouped split to avoid client overlap between training and testing data. The Decision Tree achieved 1.00 Accuracy, Precision, Recall, and F1 on the evaluated client-grouped test split while reproducing the rule-defined opportunity label. The resulting workflow produces a ranked opportunity queue intended to support human review and content-refresh prioritization rather than automatic publishing, deletion, or guaranteed CTR improvement.


## 1. Problem framing

The decision supported by this analysis is which existing content should be reviewed first for a potential content refresh.
The unit of analysis is a client-content record created by aggregating daily search-performance data for each client and content pair. The output is a ranked opportunity queue based on search visibility and engagement signals.
The main signals are impressions, CTR, and average search position. Content with high impressions, relatively low CTR, and good average position is prioritized because it has substantial visibility but comparatively low click-through performance.
A human editor can use the ranked queue to decide which content should be investigated for a possible refresh. The system does not automatically modify, publish, or delete content.
A wrong call can result in editorial time being spent on a low-value opportunity, while failing to identify a useful opportunity can leave potentially improvable content unreviewed. Data and ML help by applying the same prioritization logic consistently across a large number of content records.

## 2. Data safety

The analysis uses the FlyRank internship warehouse, with the main performance source being `fact_content_daily_performance`. Daily records were aggregated to the client-content level using total impressions, total clicks, and impressions-weighted average position, with CTR calculated from clicks divided by impressions.
The final validated dataset contains 309,234 client-content records. The main modeling features are `impressions`, `ctr`, and `avg_position`.
Records with missing impressions, clicks, or average-position values were excluded, and records with zero impressions were excluded because CTR could not be meaningfully calculated for them.
Pseudonymous `client_hash_id` and `content_hash_id` identifiers were retained only for grouping, aggregation, ranking, and traceability. They were not used as model features.
Potential leakage was considered because the opportunity target is defined using the same performance variables used as model inputs. Therefore, the model should be interpreted as reproducing the existing rule-defined label rather than independently predicting future CTR improvement.
No client names, private URLs, private search queries, or other client-identifying information are included in the analysis outputs or report.

## 3. Baseline

The transparent baseline is the rule developed earlier in the internship:
- impressions >= 237
- CTR < 0.003026
- average position <= 20
Records satisfying all three conditions are classified as opportunities with the reason code `LOW_CTR_GOOD_POSITION`.
This is a fair baseline because the capstone decision is directly based on these observable search-performance signals. It is also transparent and easy for an editor to understand and reproduce.
On the evaluated test data, the rule-defined baseline and the Decision Tree produced the same opportunity labels. The model therefore reproduced the baseline rather than demonstrating independent predictive improvement over it.
The opportunity rule identifies 66,762 opportunities out of 309,234 validated client-content records, corresponding to an opportunity rate of 21.59%.

## 4. Model / analysis

A Decision Tree classifier was used to reproduce the rule-defined opportunity label.
The exact model settings were:
- Model: Decision Tree
- `max_depth=4`
- `class_weight="balanced"`
- `random_state=42`
The feature list was:
- `impressions`
- `ctr`
- `avg_position`
Pseudonymous client and content identifiers were deliberately excluded as model features because they identify groups rather than describe the underlying performance signals.
The target is the rule-defined opportunity label: a record is classified as an opportunity when impressions are at least 237, CTR is below 0.003026, and average position is at most 20.
The model is therefore best understood as a compact learned representation of the existing opportunity rule, not as an independent model of future CTR improvement.

## 5. Evaluation

The workflow was evaluated using both a random 80/20 content-level split and a stronger client-grouped validation split.
For the client-grouped evaluation, `GroupShuffleSplit` was used with `test_size=0.20` and `random_state=42`. The resulting split contained 241,240 training rows and 67,994 test rows, with 53 clients in training and 14 clients in testing. Client overlap between the two groups was zero.
On the evaluated splits, the Decision Tree achieved:
| Evaluation | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| Week 5 — Random 80/20 | 1.00 | 1.00 | 1.00 | 1.00 |
| Week 6 — Client Grouped | 1.00 | 1.00 | 1.00 | 1.00 |
The important interpretation of these results is that the model closely reproduced the deterministic rule used to define the target. The perfect scores should therefore not be treated as evidence that the model independently predicts future CTR improvement.
A small error observed during the earlier random-split evaluation involved one record where the actual rule-defined opportunity label was 1 but the Decision Tree prediction was 0. This illustrates that the model is approximating a deterministic rule rather than providing an independent outcome forecast.
The opportunity base rate in the final validated dataset is 21.59%, with 66,762 opportunity records out of 309,234 total records.

## 6. Interpretation

The analysis found a focused subset of content that combines relatively high search visibility with relatively low CTR and good average search position.
The final validated dataset contains 309,234 client-content records, of which 66,762, or 21.59%, are classified as opportunities. This means the workflow narrows the review workload to a smaller subset rather than requiring every content record to be reviewed.
The Decision Tree's feature importance results from the earlier model analysis were:
- Impressions: 0.467488
- CTR: 0.281344
- Average position: 0.251168
In plain terms, impressions contributed the largest share of the model's rule-reproduction behavior, followed by CTR and average position.
The main negative result is that the perfect classification metrics do not demonstrate independent predictive power or causal impact. Because the target itself is constructed from the same three features, the model's strong performance is expected and should be interpreted as successful rule reproduction.

## 7. Recommendation

The recommended action is to prioritize content for human review using the following ranking logic:
1. Higher impressions are prioritized first because they represent greater search visibility.
2. Lower CTR is prioritized because it indicates comparatively fewer clicks despite that visibility.
3. Better average position is prioritized because the content is already appearing in a useful search position.
The primary recommendation is:
**Review high-impression, low-CTR content with good average position first for a potential content refresh.**
The Week 7 action queue uses the recommendation code `REVIEW_FOR_REFRESH` and the reason code `HIGH_IMPRESSIONS_LOW_CTR_GOOD_POSITION`.
A FlyRank editor could use the queue as a starting point for deciding which content deserves investigation. Human review remains required before any refresh or other editorial action.
Confidence is high that the workflow reproduces the defined opportunity rule on the evaluated validation data. Confidence is lower for any claim about future CTR improvement or business impact because no causal experiment was conducted.

## 8. Reproducibility

The analysis was developed using Python with DuckDB and the Hugging Face dataset interface. The data source is the FlyRank internship warehouse hosted on Hugging Face.
The modeling workflow uses `random_state=42`, and the Decision Tree uses `max_depth=4` with balanced class weights.
The client-grouped validation uses `GroupShuffleSplit` with `test_size=0.20` and `random_state=42`, producing 241,240 training rows and 67,994 test rows with zero client overlap.
The Week 7 workflow also generated the following reproducibility artifacts:
- `work/outputs/week7_metrics.json`
- `work/figures/week7_opportunity_distribution.png`
- `work/figures/week7_top10_action_queue.png`
The metrics artifact records the validated row count, opportunity count, opportunity rate, ranking method, and human-review/automation policy.
The workflow should be interpreted as reproducible from the committed notebooks and documented settings. The reported metrics should be regenerated from the repository workflow when performing a fresh verification rather than treated as a guarantee of unchanged results after changes to source data or aggregation logic.

The workflow is organized as a set of Jupyter notebooks in the repository, covering data access and preparation, baseline scoring, model development, validation, and the final action playbook.
The Week 7 data-access workflow uses DuckDB and Hugging Face dataset access. Required packages include `duckdb` and `huggingface_hub`, which are installed in the notebook when needed. Hugging Face access requires a read token supplied securely through a Colab Secret, environment variable, or secure prompt; no access token is stored in the repository.
The Decision Tree configuration uses `max_depth=4`, `class_weight="balanced"`, and `random_state=42`. Client-grouped validation uses a 20% test split with `random_state=42`, and client overlap between training and test sets was checked to be zero.
The capstone notebook documents the data preparation, baseline rule, model configuration, validation results, claim framing, and generated metrics and figure artifacts used for the final analysis.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset. Data and internship context credited to [FlyRank](https://flyrank.ai).


