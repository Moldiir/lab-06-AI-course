# REPORT: Testing the Tulpar Scholarship Screening Agent

## 1. What I Measured

I tested a scholarship screening agent using Qwen/Qwen2.5-3B-Instruct with greedy decoding.

The baseline experiment contained 200 matched pairs (400 applicant profiles). I tested three attributes:

- Name language.
- Gender-coded names.
- University background.

I also used two null comparisons (`null_name` and `null_university`) to estimate how much variation could occur without changing the attribute being tested.

The results were:

| Attribute | Pair flips | Flip rate | Mean absolute score difference |
|---|---:|---:|---:|
| Name language | 2/40 | 5.0% | 3.12 |
| Gender-coded name | 4/40 | 10.0% | 1.12 |
| University | 1/40 | 2.5% | 2.50 |
| Null name | 0/40 | 0.0% | 2.00 |
| Null university | 1/40 | 2.5% | 2.12 |

The results show some differences between matched applicants. However, the sample was small, and the null comparisons also produced score differences. Therefore, I cannot conclude that every observed difference was caused by the tested attribute.

For gender-coded names, all four decision flips went against the male-coded name. The shortlist rate was 80% for male-coded names and 90% for female-coded names. However, the McNemar test gave p = 0.125, so this result was not statistically significant.

For university background, flagship-university profiles received an average score advantage of 2.5 points over regional-university profiles. All 10 pairs with different scores favoured the flagship university. However, only one pair changed its final decision, and the score difference was close to the null comparison. This suggests a possible score-level effect, not proof of unequal treatment.

In the baseline run, 356 of 400 profiles were shortlisted and 44 were rejected. The model produced no unparseable responses.

## 2. What I Found

The agent shortlisted most applicants. This is important because a system that accepts almost everyone may not distinguish applicants effectively.

Most scores were far from the decision threshold. However, in the 40–49 score range, 31 applicants were shortlisted and 33 were rejected. This shows that decisions near the threshold deserve particular attention.

The experiment also showed that a small number of matched pairs can produce different decisions. However, the sample size limits the strength of the conclusions. A result that is not statistically significant does not prove that discrimination is absent.

The null comparisons are important because they show the amount of variation that can occur even when the tested attribute is not changed.

## 3. Who Is Affected

The main people affected are scholarship applicants.

A shortlist or rejection recommendation can influence whether an applicant receives further consideration and potentially access to financial support.

Applicants may be affected by:

- Their name or the language associated with it.
- Their gender-coded name.
- Their previous university.
- Information used as a proxy for protected characteristics.
- Changes in the model's prompt or scoring behaviour.

The results suggest that university background and name-related information deserve monitoring. However, the experiment does not establish that the model intentionally discriminates against any group.

Applicants with borderline scores are also affected because they may be sent for additional human review. Applicants outside the review range may receive an automatic recommendation without the same additional attention.

## 4. What the Harness Changed

I implemented five active controls: input validation, blinding, refusal, decision logging, and human escalation.

The blinding control hides the applicant's name and university from the model.

After blinding, the two members of each matched pair received identical model-visible text. Therefore, pair flips and score differences became zero for all five attributes. This is zero by construction, not proof that the model became fair or stopped relying on those attributes. Other information may still act as a proxy.

The individual results changed:

- 36 of 400 profiles (9.0%) received a different outcome.
- 17 changed from shortlist to reject.
- 12 changed from shortlist to escalate.
- 5 changed from reject to shortlist.
- 2 changed from reject to escalate.
- Shortlisted profiles decreased from 356 to 332.
- Rejected profiles increased from 44 to 54.
- Scores changed for 166 of 400 profiles.
- The mean absolute score change was 14.01 points.

The prompt changed when names and universities were removed. Therefore, these changes do not prove that the model relied on those fields in the baseline run. They do show that the model was sensitive to changes in prompt structure.

### Human Escalation

The harness sent 14 of 400 profiles (3.5%) to a human reviewer.

Eight were escalated because of the borderline score rule, and six because of the reason-word rule.

The planned review share was 20% (80 of 400), but the actual share was much lower. The score band (45, 55) had been calibrated using the baseline scores and was not recalibrated after blinding changed the prompt and scores.

The reason-word rule also produced at least one confirmed false alarm. The model used the phrase "strong academic background" in a positive academic context, but the word "background" triggered escalation.

This shows that keyword matching can misunderstand context and create unnecessary review work.

### Refusal and Validation

I tested six requests through the bare pipeline and the harness.

The harness refused five requests without making a model call:

- An inappropriate screening purpose.
- An applicant under 18.
- Missing consent.
- Invalid numerical values.
- Unexpected instructions in the name field.

One ordinary request received a decision.

The 400 clean profiles in the main experiment produced zero refusals. This does not prove that the refusal rules will never reject legitimate applicants. For example, unusual names or programme titles may trigger the free-text rule.

### Cost

The harness made 400 model calls before blinding and 400 after blinding: one call per profile in each run.

The counterfactual check was disabled (`counterfactual_check=False`), so it did not add model calls.

| Measurement | Before blinding | After blinding |
|---|---:|---:|
| Model calls | 400 | 400 |
| Prompt tokens | 84,409 | 77,922 |
| Runtime | 474.7 seconds | 473.8 seconds |

The harness did not significantly increase runtime in this experiment.

## 5. Where My Responsibility Lies

My responsibility is not only to implement controls but also to understand their limitations and explain their effects.

The experiment showed that blinding removes names and universities from the model's input, but it does not prove that the final decisions are fair. The prompt change also affected scores and outcomes, so I cannot claim that every change was caused by removing potentially sensitive information.

The borderline score band needs to be recalibrated using the scores of the final system configuration. Otherwise, the intended human review rate may not be achieved.

The reason-word rule needs better context handling because it can create false alarms. The refusal rules should also be tested on legitimate names and programme titles to identify incorrect refusals.

The decision log supports basic auditing, but it does not record the human reviewer's final decision. This should be added so that the organisation can evaluate whether human review corrects mistakes.

The counterfactual check was disabled, so I cannot use it as evidence that the final system passed an additional fairness test.

My conclusion is that the harness makes the system more controlled and easier to inspect, but it does not guarantee fair scholarship decisions. Before real deployment, the system would need further testing, recalibration, better human-review records, and regular monitoring.
