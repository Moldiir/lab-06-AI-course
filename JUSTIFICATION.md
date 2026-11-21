JUSTIFICATION: What the System May Do, What It Must Refuse, and What It Records
The scholarship screening agent can be biased or react to information that is not related to an applicant's academic merit. I implemented five active controls: input validation, blinding, refusal, decision logging, and human escalation.

The counterfactual check exists in the notebook but was disabled in the submitted run (counterfactual_check=False). With names and universities already hidden, changing those fields would not change the text received by the model. The controls reduce some risks and make decisions easier to review, but they do not guarantee fairness.
1. What the System May Do
The system may process one consenting adult applicant's profile for the stated purpose of scholarship screening.

It can:
- Read academic and other permitted application information.
- Evaluate an applicant using the visible merit-related fields.
- Produce a shortlist or rejection recommendation with a score.
- Send uncertain cases to a human reviewer.
The model does not receive the applicant's name or university. These fields are hidden through the blinding control.

Household income remains visible because it is treated as a relevant scholarship need criterion.

The system uses a borderline score range of 45–55. Cases in this range are sent to a human reviewer instead of being treated as ordinary automatic decisions.
2. What the System Must Refuse
The system must refuse requests that do not meet the required conditions.

The refusal and validation controls cover:
- A purpose other than scholarship screening, such as ranking applicants by ethnicity.
- Applicants under 18.
- Applicants who have not consented to automated processing.
- Invalid numerical values, such as GPA 9.9 or an exam score of -5.
- Unexpected free-text input, such as an instruction placed in the name field.
The refusal test contained six requests. Five were refused without a model call, and one ordinary request received a decision.

The decision log contains five refusal records with the following reason codes:
- purpose_not_allowed
- minor
- no_consent
- invalid_input
- free_text_unexpected_characters
The bare pipeline processed all six requests and returned a verdict for each. This demonstrates why the refusal control is necessary.

However, the free-text rule has a limitation: it may reject legitimate applicants whose names or programme titles contain characters that the rule does not expect. A refusal also ends the request without automatically sending it to a human reviewer.
3. What Record the System Keeps
The file decisions.jsonl contains 406 records:
- 400 records from the main experiment.
- 6 records from the separate refusal test.
The log records:
- Record ID and application ID.
- A SHA-256 hash of the exact text shown to the model.
- Fields shown and fields withheld.
- Model ID and prompt version.
- Harness settings.
- Raw model output.
- Decision and score.
- Controls that were activated.
- Final outcome and reason code.
- Whether the case was sent to a human.
- Timestamp.
The log stores field names and a hash rather than the complete applicant profile.

The submitted log confirms that names and universities were withheld from the model. It also confirms that the counterfactual check was disabled.
Limitations of the Logging System
The logging design has several weaknesses:
1. It records that a case was sent to a human, but it does not record the human reviewer's final decision.
2. The original text shown to the model cannot be fully reconstructed from the hash alone.
3. The file is append-only in the current workflow but does not use a hash chain, so modifications could go unnoticed.
4. The five refusal test records reuse the PID P0124a and are not explicitly marked as test records.
5. The raw_output field may contain text that was shown to the model.
These limitations mean that the log supports basic auditing but is not a complete or tamper-evident audit system.
4. Trade-offs I Accepted
Blinding
Hiding names and universities removes these two fields from the model's input. However, blinding also changes the prompt.

After blinding:
- 36 of 400 profiles changed their outcomes.
- 166 of 400 profiles had different scores.
- The mean absolute score change was 14.01 points.
The result shows that removing two fields changed the model's behaviour substantially. It does not prove that the model relied on those fields before blinding, because the prompt itself also changed.

Blinding also cannot remove every possible proxy for protected characteristics.
Borderline Score Band
I selected the score band (45, 55) using the baseline results.

The baseline comparison showed:
Score band	Profiles sent to review	Percentage
(45, 60)	163/400	40.8%
(40, 50)	109/400	27.3%
(45, 55)	80/400	20.0%
(45, 45)	34/400	8.5%


I selected (45, 55) as a compromise between reviewer workload and review coverage.

However, after blinding, only 8 of 400 profiles were escalated by the score band. The band was not recalibrated after the prompt changed.

Another limitation is that the band focuses on borderline scores. Applicants with low scores, including rejected applicants scoring around 40, may not receive additional human review.
Reason-Word Escalation Rule
The reason-word rule was intended to identify explanations that might refer to protected characteristics.

It caused 6 of the 14 escalations. At least one of them was a confirmed false alarm: the model wrote "strong academic background", and the word "background" triggered a human review even though the phrase was used in a positive academic context. I did not manually verify all 6 cases, so I cannot claim that all of them were false alarms.

This shows that keyword matching does not understand context. It can increase reviewer workload without identifying actual discrimination.
Counterfactual Check
The counterfactual check was disabled in the submitted run.

The system already hides names and universities. Therefore, changing these fields would not change the model-visible text, and the comparison would be uninformative.

The check would also require additional model calls. I did not count it as an active control in the final experiment.
Human Escalation
Human escalation provides an opportunity to review uncertain decisions rather than relying entirely on the model.

However, the system only records that a case was escalated. It does not record the human reviewer's final decision. This limits accountability and makes it difficult to evaluate whether human review corrected the model's mistakes.
5. Predictions and Observations
Prediction 5 — Written Before Reading the Laws
I expected the hardest control to connect to a specific legal provision to be blinding.

I did not expect a law to directly require hiding an applicant's name or university from an AI model. I expected the legal connection to be indirect, through principles such as non-discrimination and collecting only necessary personal data.

I also expected the free-text refusal rule to be difficult to connect to a specific legal article because prompt injection and unexpected instructions are technical security problems.

I expected logging and human escalation to be easier to justify because legal instruments contain provisions related to accountability, records, and human intervention.
Observation
The experiment confirmed that technical controls do not always have a direct one-to-one legal equivalent. Some controls are engineering choices designed to support broader legal duties.

Blinding supports the goals of fairness and data minimisation, but it does not by itself prove compliance or eliminate discrimination.

Human escalation and logging have clearer connections to accountability and human oversight. However, the actual implementation still has limitations because human review outcomes are not recorded.
6. Legal Basis
The following legal instruments provide the legal context for the controls. Their applicability depends on the organisation operating the scholarship system, the location of the applicants, and the processing activities.
Kazakhstan: Law on Personal Data and Their Protection
Article 7 — Conditions for Collection and Processing of Personal Data

Article 7 establishes consent of the data subject or their legal representative as the general condition for collecting and processing personal data. However, it also provides exceptions, including the cases specified in paragraph 5 of Article 7 and Article 9 of the Law. For my prototype, this means that applicant consent should be obtained before processing personal data unless a specific legal exception applies. Consent should not be treated as the only possible legal basis in every situation.

It is relevant to the refusal control because the prototype refuses applicants who have not consented to automated processing. This is the prototype's chosen rule and should not be interpreted as a statement that consent is the only legal basis permitted by the Law.

Article 11 — Confidentiality of Personal Data

This article requires protection of personal data and restricts its distribution without consent or another lawful basis.

It supports limiting access to applicant information and hiding unnecessary fields from the model.
Kazakhstan: Constitution of the Republic of Kazakhstan
Article 14 — Equality and Non-Discrimination

This article establishes equality before the law and prohibits discrimination on grounds including gender, race, nationality, language, place of residence, and other circumstances.

It is relevant to fairness testing because the experiment compares outcomes for applicants with different names and university backgrounds.
European Union: General Data Protection Regulation (GDPR)
The GDPR is relevant as a benchmark and may apply when the processing falls within its territorial scope.

Article 5 — Principles Relating to Processing of Personal Data

The principles include data minimisation, accuracy, and accountability. These principles support limiting the information shown to the model and keeping records of processing decisions.

Article 22 — Automated Individual Decision-Making

This article applies to decisions based solely on automated processing that produce legal effects or similarly significantly affect individuals. It provides safeguards in applicable cases, including the right to obtain human intervention, express one's point of view, and contest the decision.

Sending only borderline or flagged applications to a human reviewer does not automatically guarantee meaningful human intervention for every applicant. If Article 22 applies to this scholarship decision, the system would need to provide the required safeguards to affected applicants. My prototype does not establish that these legal requirements are fully satisfied.

It is relevant to human escalation because scholarship decisions may significantly affect applicants, but the current escalation mechanism alone should not be treated as proof of compliance with Article 22.
European Union: Artificial Intelligence Act
The AI Act is used here as a regulatory benchmark. Its applicability depends on its scope and the system's intended use.

Article 14 — Human Oversight

This article requires effective human oversight for high-risk AI systems. It supports the decision to send uncertain cases to a human reviewer.

Article 12 — Record-Keeping

This article addresses automatic logging capabilities for high-risk AI systems. It is relevant to the decision logging control and the need for records that support monitoring and review.
Legal Applicability
The Kazakhstan Constitution and the Law on Personal Data and Their Protection are the primary national legal references for a system operated in Kazakhstan, subject to the applicable legal requirements.

The GDPR and EU AI Act are used as comparative benchmarks unless the system or organisation falls within their respective scopes.

These instruments support the goals behind the controls, but they do not mean that this experimental notebook is automatically legally compliant.
7. AI-Use Declaration
I used Claude (Anthropic)
