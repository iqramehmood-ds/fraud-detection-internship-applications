# Fraud Detection in Internship Applications

Data Analytics Internship — Task 4 (Major Project)

## Objective

Identify anomalies in internship applications to catch fake or manipulated entries before they reach the review stage, instead of relying on a single fixed rule.

## Dataset

No dataset was provided for this task, so a synthetic dataset of 1,127 internship applications was generated, with fields similar to a real application form: name, email, phone, IP address, university, department, GPA, age, graduation year, cover letter length, and submission timestamp.

950 records represent genuine applications. Three fraud patterns were deliberately injected so the detection methods could be evaluated against a known ground truth:

- **Duplicate identities** (81 records) — the same applicant re-applying under a slightly changed name or new email, often from the same IP, within minutes of the first submission.
- **Rapid / bot-style submissions** (66 records) — bursts of 6–12 applications from the same IP within seconds to a couple of minutes, with very short, generic cover letters.
- **Inconsistent data** (30 records) — GPA outside the 0–4 range, implausible ages, or graduation years that don't make sense.

## Approach

1. **Rule-based checks** — exact duplicate email/phone, same-IP submission bursts and their timing, GPA/age/graduation-year validity, cover letter length.
2. **Isolation Forest** — scores every application on how easily it isolates from the rest of the data; isolated points are more likely to be anomalies.
3. **K-Means clustering** — groups applications into clusters (k chosen via the elbow method) and flags clusters that are unusually small or unusually scattered as suspicious.
4. **Ensemble alert system** — an application is flagged if at least two of the three methods agree, or if a hard rule fires on its own (e.g. an exact duplicate email or an impossible GPA). Each alert is logged with a plain-language reason.

## Results

| Method | Precision (fraud) | Recall (fraud) | F1-score |
|---|---|---|---|
| Isolation Forest | 0.796 | 0.814 | 0.804 |
| K-Means (outlier clusters) | 0.816 | 0.927 | 0.868 |
| **Combined alert system** | **0.821** | **0.960** | **0.885** |

The combined system flagged 207 of 1,127 applications (18.4%), catching 170 of the 177 injected fraud cases while flagging 37 genuine applications for manual review.

## Files

- `fraud_detection.ipynb` — full pipeline: data generation → feature engineering → Isolation Forest & K-Means → ensemble alert system → visualizations
- `applications.csv` — the generated dataset
- `flagged_applications_alert_log.csv` — the applications that triggered an alert, with reasons
- `alert_report.txt` — human-readable version of the alert log

## Tech stack

Python, pandas, NumPy, scikit-learn (Isolation Forest, K-Means, PCA), rapidfuzz, matplotlib, seaborn

## Notes / limitations

- This is a synthetic dataset built for the task, so the fraud patterns are cleaner than what a real application pool would show.
- The system is designed to flag applications for manual review, not auto-reject them — a few genuine applicants can legitimately share an IP (e.g. a university lab) or apply close together in time.
- Isolation Forest's contamination rate was set from what the rule-based checks were already catching; in a live system it would need tuning as real fraud rates become clearer.
