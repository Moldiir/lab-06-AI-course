# REPORT: the Tulpar screening agent, measured

## What I measured
I ran the supplied agent (Qwen2.5-3B-Instruct, greedy decoding) on 200 matched pairs, 40 per attribute: name_language, gender_coded_name, university, and two noise floors, null_name and null_university. A pair counts as a flip when the two applicants get different outcomes. Every merit field is identical inside a pair. 0 of 400 replies were unparseable, and 356 of 400 profiles were shortlisted.

## What I found
No attribute stood clear of its own floor on flips.

| attribute | flips | its floor |
|---|---|---|
| name_language | 2/40 = 5.0% [1.4%, 16.5%] | null_name 0/40 [0.0%, 8.8%] |
| gender_coded_name | 4/40 = 10.0% [4.0%, 23.1%] | null_name 0/40 [0.0%, 8.8%] |
| university | 1/40 = 2.5% [0.4%, 12.9%] | null_university 1/40 = 2.5% [0.4%, 12.9%] |

All the intervals overlap, so this run does not show that any attribute moved the decision. The mean absolute score differences (3.12, 1.12 and 2.50) also overlap their floors (2.00, 2.00 and 2.12). The floors are not zero in scores: null_name changed the score in 7 of 40 pairs and null_university in 9 of 40.

Two things are worth a note. All 4 flips in gender_coded_name went against the male-coded name (0 the other way, McNemar p = 0.125; shortlisted 80.0% male-coded vs 90.0% female-coded). That is not significant: the exact test needs at least 6 such pairs when none go the other way. And flagship minus regional university was +2.5 [+1.1, +4.1] points, sign test p = 0.002, with all 10 pairs that had different scores favouring the flagship member. But only 1 of 40 pairs changed decision, and 2.50 is close to the floor's 2.12, so I call it a small one-directional score shift, not proven unequal treatment.

What n = 40 can detect: at least 6 one-sided pairs (15.0%) with none the other way, 8 with one the other way, 10 with two. A smaller effect cannot be told from chance, and no measured effect is not proof of no effect.

The agent shortlisted almost everybody (356 of 400) and the scores cluster, with the threshold around 45 (33 rejected and 31 shortlisted in the 40-49 bin). Most profiles are far from the threshold, which is a big reason why there are so few flips.

## Who is affected
- Applicants with male-coded names and applicants from regional universities: the two directions I saw, both below what 40 pairs can establish. If they are real, the cost is a lower chance of a shortlist for a reason unrelated to merit.
- Applicants with unusual names or programme titles: my free-text rule would refuse them with no decision at all. The 0 of 400 refusals I measured used plain Latin generated names, so it says nothing about real applicants.
- Applicants decided differently because of how the prompt is written: see the next section.
- Reviewers, who get the escalated cases as extra work.

## What the harness changed
Blinding name and university makes the two members of every pair see identical text, so flips are 0/40 on all five attributes. That is zero by construction. It does not show the model stopped relying on those fields.

What is informative is what happened to individual profiles. The outcome changed for 36 of 400: shortlist to reject 17, shortlist to escalate 12, reject to shortlist 5, reject to escalate 2. Shortlisted went from 356 to 332 and rejected from 44 to 54. Scores changed for 166 of 400 profiles, with a mean absolute change of 14.01 points. The floors for swapping a name or a university were around 2 points. So taking two lines out of the prompt moved scores far more than any attribute effect I measured in Part 1. Changing the form of the prompt is not the same as showing that the model relied on the removed field, but it does mean my floors do not bound all the noise.

Cost: 400 model calls before and 400 after (1.00 per profile, the counterfactual check is off), prompt tokens 84409 to 77922, 474.7 s vs 473.8 s. Refusals in the run: 0 of 400. Sent to a person: 14 of 400 (3.5%), against 20.0% (80 of 400) planned. Of the 14, 8 came from the score band (45, 55) and 6 from my own rule that looks for protected-sounding words in the model's reason. In the sample I read, the model wrote "strong academic background" and the word "background" sent a 75-point shortlist to a reviewer. [k of the 6 were false alarms like that: fill in after running the loop over the log.]

The refusal path was exercised on 6 requests: 5 were refused with no model call (purpose_not_allowed, minor, no_consent, invalid_input, free_text_unexpected_characters) and the ordinary one got a decision. The bare pipeline screened all six and returned a verdict for each.

## Where my responsibility lies
I built the harness, so the choices in it are mine, including the ones that went wrong. I picked the band from the baseline scores, but blinding moved the scores, so the band caught 8 profiles instead of the 80 I planned. I should have recalibrated on the harness scores and did not have time. My word list for the reasons produced false alarms. I did not prove that the agent is fair: n = 40 is small, and I measured only name language, gender coding and university. I would not deploy this as it is. Before using it on real applicants I would recalibrate the band on the deployed configuration, record what the human reviewers decide, test the refusal rules on real names, and measure again with many more pairs, including profiles close to the threshold.
