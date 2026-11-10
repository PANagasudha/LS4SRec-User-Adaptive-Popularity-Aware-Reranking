# LS4SRec: User-Adaptive Popularity-Aware Re-ranking

## Architectural Execution

This document explains the execution architecture of the project, starting from the input dataset and trained LS4SRec outputs and ending with the final Top-K recommendations and evaluation.

## 1. Overall Execution Flow

```text
Amazon Home & Kitchen Interaction Data
                  |
                  v
          Data Preparation
                  |
                  v
       User Interaction Sequences
                  |
                  v
       Pre-trained LS4SRec Model
                  |
                  v
       Original User-Item Scores
                  |
                  v
      Popularity and User-Preference
             Calculation
                  |
                  v
       Popularity-Aware Re-ranking
          |       |       |
          |       |       |
         E1      E2      E3 / E4
          \       |       /
           \      |      /
                  v
        Re-ranked Top-K Lists
                  |
                  v
       Recommendation Evaluation
                  |
                  v
        Metrics and Results
```

## 2. Input Data

The project uses the **Amazon Home & Kitchen** interaction dataset.

The interaction records are used to prepare sequential user histories.

The notebook expects the following dataset files:

- `all_train_seq.txt`
- `home.txt`

The training interactions are used to calculate item popularity and each user's historical popularity preference.

## 3. LS4SRec Recommendation Model

The project uses LS4SRec as its base sequential recommendation model.

LS4SRec learns user preferences from interaction sequences by modelling:

- Long-term user interests
- Short-term user interests

The model generates an original recommendation score for each candidate item.

For a user `u` and candidate item `v`, let:

\[
s_{u,v}
\]

represent the original LS4SRec recommendation score.

**Execution note:** The supplied extension notebook loads previously trained model outputs from `model_outputs.npz`. It performs the re-ranking and evaluation stages. It does not retrain LS4SRec when the notebook is executed.

## 4. Popularity Calculation

The project calculates two important values using training interactions.

### 4.1 Item Popularity

Item popularity is based on how frequently an item appears in the training interactions.

A normalized log-popularity value is calculated:

\[
q_v
\]

where `q_v` represents the normalized popularity of item `v`.

### 4.2 User Popularity Preference

For every user, the mean popularity of items in their training history is calculated:

\[
q_u
\]

This value represents the popularity level of items that the user has interacted with previously.

## 5. Popularity-Aware Re-ranking

After LS4SRec generates the original recommendation scores, the project adjusts them using a popularity penalty.

The re-ranking formula is:

\[
s'_{u,v}=s_{u,v}-\beta_u q_v
\]

Where:

- `s(u,v)` = Original LS4SRec recommendation score
- `q(v)` = Normalized popularity of candidate item
- `β(u)` = Popularity penalty strength for the user
- `s'(u,v)` = Adjusted recommendation score

The adjusted scores are used to rank the candidate items.

Highly popular items receive a larger penalty, which can improve the ranking position of less-popular items.

## 6. Re-ranking Strategies

The notebook implements and compares six strategies.

| Strategy | Execution |
|---|---|
| E1 – Original LS4SRec | Uses original LS4SRec scores without a popularity penalty. |
| E2 – Fixed Penalty | Applies the same popularity penalty to every user. |
| E3 – User-Adaptive Penalty | Adjusts the penalty according to the user's mean historical item popularity. |
| E3-rank | Uses a rank/percentile-based representation of the user's popularity preference. |
| E3-rank-shrink | Shrinks rank-based user estimates toward the average for users with short histories. |
| E4 – Recent-Interest Variant | Combines full-history and recent-history popularity preference, using the last three items for recent history. |

### 6.1 Main Strategy: E3 – User-Adaptive Penalty

The user-adaptive penalty is calculated using:

\[
\beta_u=\beta_{\max}(1-q_u)
\]

Where:

- `βmax` = Maximum popularity penalty
- `q(u)` = Mean popularity of items in the user's training history
- `β(u)` = Final penalty for that user

**Execution steps:**

1. Calculate the user's mean historical item popularity.
2. Calculate the user's penalty strength using the E3 formula.
3. Calculate the popularity penalty for each candidate item.
4. Subtract the penalty from the original LS4SRec score.
5. Rank candidate items using the adjusted scores.
6. Generate the final Top-K recommendations.

A user with a lower historical popularity value receives a stronger popularity penalty.

A user with a higher historical popularity value receives a weaker popularity penalty.

## 7. Validation and Test Execution

The project uses separate validation and test stages.

```text
Original LS4SRec Scores
          |
          v
     Validation Data
          |
          v
 Select Re-ranking Strategy
   and Penalty Strength
          |
          v
      Selected Setup
          |
          v
        Test Data
          |
          v
    Final Evaluation
```

The validation selection rule aims to improve Long-tail Recall@10 while retaining at least 95% of the original E1 Recall@10.

After selecting the configuration using validation data, the test set is used for final evaluation.

## 8. Evaluation Metrics

The project evaluates recommendation quality and item exposure using the following metrics:

### Recall@10
Measures how many relevant items are retrieved in the Top 10 recommendations.

### NDCG@10
Measures ranking quality by giving higher importance to relevant items appearing near the top.

### Long-tail Recall@10
Measures recommendation performance for less-popular items.

### Coverage@10
Measures how many distinct items appear across the Top-10 recommendation lists.

### Average Popularity of Recommended Items
Measures the popularity level of the recommended items.

Paired bootstrap evaluation with 1,000 resamples is also used to compare selected strategies.

## 9. Notebook Execution Requirements

The supplied notebook is designed to run in Google Colab.

### Expected Input Structure

```text
/content/
|
|-- model_outputs.npz
|
|-- LS4SRec/
    |
    |-- datasets/
        |
        |-- home/
            |
            |-- all_train_seq.txt
            |
            |-- home.txt
```

The notebook may also use the following Google Drive location for saving experiment outputs:

```text
/content/drive/MyDrive/LS4SRec_Project/colab_results/
```

An optional `history.csv` file may also be used for the interactive demonstration.

### Execution Order

1. Mount Google Drive if it is being used for input or output files.
2. Load the Amazon Home & Kitchen dataset files.
3. Load the precomputed LS4SRec outputs from `model_outputs.npz`.
4. Calculate item popularity and user popularity preference using training interactions.
5. Execute E1, E2, E3, E3-rank, E3-rank-shrink and E4.
6. Select the configuration using validation data.
7. Evaluate the selected configuration on test data.
8. Generate metric tables and comparison results.
9. Save experiment outputs to the configured results folder.

## 10. Important Execution Notes

- The supplied notebook performs popularity-aware re-ranking using previously trained LS4SRec outputs.
- It does not perform a complete end-to-end LS4SRec training run.
- The trained model outputs and required dataset files must be supplied separately.
- The HTML demo displays prepared examples and results. It does not perform live LS4SRec model inference.
