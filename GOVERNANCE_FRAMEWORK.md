# Responsible AI Governance Framework for Public-Service Deployments

**Author:** Harsheen Kaur

**LinkedIn:** https://www.linkedin.com/in/harsheen285/

**GitHub:** https://github.com/Harsheen-Kaur-Projects

---

## Why this document exists

A lot of AI governance frameworks are written before there is much evidence of what can actually go wrong. I started this one with evidence from an actual evaluation. It is based on findings from my own [Responsible AI Evaluation Harness for Public-Service Chatbots](https://github.com/Harsheen-Kaur-Projects/public-service-ai-evaluation-harness), a working evaluation pipeline that I built and ran against a real model (`openai/gpt-oss-20b`) using 16 citizen-service queries across health, education, social protection, and disaster response.

One finding from that evaluation ended up shaping the rest of this framework.

One of the model's worst results was on an adversarial question specifically designed to expose weaknesses: `adv_safety_002`, the single lowest score in the entire run at 1.25/4. Two other low scores came from `social_001` and `social_003`, ordinary social-protection questions that hadn't been identified as particularly high-risk when the dataset was designed. Both scored 1.85/4.

The criterion-level results help explain what happened across all three. Hallucination was the weakest area, with an average score of 1.44/4, followed by factual accuracy at 2.38/4. Both were considerably weaker than safety (3.81/4) and language coverage (4.0/4).

That result changed how I think about the governance problem. Adversarial testing did catch a real failure here, so those cases clearly matter. At the same time, a process focused only on expected problems would have missed `social_001` and `social_003`, since nothing in the dataset design had flagged them as risky. Evaluation needs to be systematic and continuous, covering ordinary questions alongside deliberately difficult ones. In this run, two of the three worst failures came from the ordinary set.

My other two projects looked at different parts of the problem. The evaluation harness measures whether the AI performs reliably. My [CivicAssist](https://github.com/Harsheen-Kaur-Projects/civicassist-product-case-study) prototype looks at what the product should do when the evidence is available or when it is not.

The governance question comes next:

Once an AI system like this is deployed at scale across a large organization and its country operations, how should the organization manage the risk over time?

---

## 1. Risk Matrix

The likelihood ratings below come from the harness's live evaluation. For risks the current harness cannot measure, I have left them unscored rather than assigning a number the evaluation can't actually justify.

| Risk type                                                                  | Likelihood (evidence)                                                                                                                                                                                                                                                            | Impact                                                                                                                                                              | Mitigation                                                                                                                                                                                                                                                                                        |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hallucination (confidently stated but ungrounded details)              | High. This was the weakest of the five rubric criteria, scoring 1.44/4. It was also a major factor behind the `social_001` and `social_003` failures, which were the two lowest-scoring queries in the run despite being routine questions.                                  | A citizen could act on a fabricated eligibility rule, deadline, or contact detail and lose access to a benefit they qualify for or miss an important filing window. | Require citation grounding at the product level, with every substantive claim traceable to a source. Set a pre-deployment threshold of at least 3/4 for hallucination across the full query set, including ordinary questions. Re-run the harness regularly so evaluation continues after launch. |
| Factual inaccuracy (incorrect information, even when not fabricated)   | High. The second-weakest criterion was factual accuracy, at 2.38/4.                                                                                                                                                                                                          | The potential harm is similar to hallucination, but these errors can be harder to spot because the information may sound completely plausible.                      | Maintain versioned source-of-truth documents and flag changes to the underlying policy. CivicAssist's "Updated Mar 2026" pattern is one example. Schedule regular fact checks against current official documentation.                                                                             |
| Unsafe compliance (giving harmful advice instead of deferring)         | Medium in this run. Safety was actually the strongest criterion at 3.81/4. However, hand-graded examples showed that a 0/4 outcome is still possible when the model follows an unsafe framing instead of overriding it, for example, treating a red-flag symptom as routine. | Potentially severe. Health and disaster-response questions can involve physical safety, not just administrative consequences.                                       | Use hard-coded override rules for known red-flag situations, such as specific symptoms or emergency instructions. These should bypass the model and route the user to a fixed, human-reviewed response.                                                                                           |
| Accessibility exclusion (language, literacy, disability, connectivity) | Not fully measured. Language coverage scored 4.0/4, but the current test only covers English, Spanish, and French. Literacy level, screen-reader compatibility, and low-connectivity performance are not currently measured.                                                 | High at population scale. Accessibility problems often do not appear as obvious failures; they can simply prevent certain groups from using the service.            | Expand language testing to reflect each country's actual official and locally relevant languages. Make plain-language and low-bandwidth modes part of the pre-deployment checklist.                                                                                                               |
| Bias / inconsistent treatment across demographic framing               | Not currently measured. The existing rubric and dataset do not include paired queries that isolate this issue. This is an evaluation gap, not evidence that the risk is low.                                                                                                 | Potentially severe. Inconsistent treatment based on who is asking can damage trust in the institution as well as the tool.                                          | Create paired adversarial tests using the same underlying question with different demographic framing. For systems involved in eligibility or entitlement decisions, the absence of bias testing should be treated as a deployment blocker.                                                       |
| Privacy / data handling                                                | Not currently measured. Privacy is not covered by the current rubric.                                                                                                                                                                                                        | High. Public-service interactions can involve income, health, immigration, family circumstances, and other sensitive information.                                   | Add privacy and data handling to the evaluation framework before live deployment. Require a separate review of data retention, access controls, and logging so these issues receive their own review.                                                                                             |

Bias and privacy are left unscored above because the current rubric doesn't test for them. This is a measurement gap, not evidence that the risk is low, and closing it is a precondition for any real deployment involving eligibility decisions.

---

## 2. Governance Framework

### 2.1 Pre-deployment risk assessment

Before a model or prompt configuration is approved for use in a country office, the full harness needs to be run against it. The test needs to cover the difficult cases as well as the ordinary questions people are likely to ask.

Deployment requirements:

* A minimum score for each individual rubric criterion, not just an overall average. The live evaluation showed why this matters: a model can look acceptable overall while still performing poorly on one important dimension, as happened with hallucination.

* Confirmation that the query set has been adapted to the country's actual languages, benefit programs, and regulatory environment. Each deployment needs a localized test set rather than a direct copy from another deployment.

* A completed privacy and data-handling review conducted separately from the quality evaluation, since privacy is not currently covered by the rubric.

### 2.2 Live monitoring

After deployment, a sample of real production queries and responses can be logged and periodically evaluated using the same rubric used before launch.

The evaluation needs to continue because the underlying information can change. A response that was correct when the system launched may become wrong later. A scholarship rule updated in March, for example, could invalidate an answer that was previously accurate.

Poor-scoring responses or responses that trigger predefined risk flags go to human review before they pass without intervention. This connects directly to CivicAssist's uncertainty state: when the evidence is insufficient, the system should be able to say so instead of filling the gap with a guess.

### 2.3 Human escalation path

Any response flagged during monitoring, along with any response that a citizen reports as incorrect or unhelpful, goes to a named human reviewer with a defined response-time target.

An escalation queue needs a named owner to provide meaningful human oversight. CivicAssist's "not fully covered by our sources" state has the same requirement. Telling a user the system is uncertain only helps if there's somewhere for that uncertainty to go.

### 2.4 Post-deployment audit cycle

The full evaluation harness needs to be run against the live system on a fixed schedule. Quarterly is a reasonable starting point, with an additional evaluation whenever an underlying policy, eligibility rule, or other important source changes.

Results need to be recorded internally for each deployment and compared with its original baseline.

Any decline in an individual criterion warrants investigation, even if the overall average still looks acceptable. An aggregate score can hide a regression in one area, which is exactly the kind of issue this evaluation brought to light.

---

## 3. Country-Context Considerations

Organizations operating across many countries, particularly international development agencies, face very different conditions around connectivity, digital literacy, language, and regulation.

Governance needs to be adjusted when the operating environment changes.

### Low-connectivity settings

Live monitoring assumes that the system can reliably record and transmit data. That assumption may not hold in lower connectivity environments.

In those settings, monitoring may need a lightweight, batch-based logging approach. The deployment checklist also needs to define what happens when the system cannot reach its logging backend, alongside what happens when the model has low confidence.

### Low-literacy populations

The escalation model also relies on users being able to recognize when an answer is wrong and report it.

That assumption may not hold everywhere. Where digital or general literacy is limited, the system may need to escalate more conservatively on its own. For example, lower confidence thresholds could trigger human review instead of depending on users to identify and report incorrect answers.

### Varying regulatory and data-protection regimes

Privacy requirements vary between countries, so a single global rule will not cover every deployment.

Requirements around consent, data retention, access, and cross-border data handling vary between countries. The privacy review should therefore be completed for each deployment against the relevant local requirements. Approval from an earlier deployment therefore needs to be reviewed again.

These considerations do not cover every country-specific issue. They provide a starting point for governance that accounts for the environment in which the system is actually being used.

---

## 4. Related Frameworks

Existing work in trustworthy and responsible AI provides the broader foundation for this document. Several major institutions have already established guidance in this area, while the discussion here applies that thinking to a specific public-service use case and the failures observed in the evaluation.

The **OECD AI Principles**, originally adopted in 2019 and updated in 2024, provide an intergovernmental framework for trustworthy AI based on a risk-based and lifecycle-oriented approach. Their principles cover areas including inclusive growth, human rights, transparency, robustness and safety, and accountability.

The pre-deployment, monitoring, and audit structure used here is consistent with that lifecycle approach. The scope is narrower: the OECD Principles are broad and jurisdiction-agnostic, while this framework focuses on a specific public-service use case and starts with observed model failures.

The **United Nations Development Programme (UNDP)** provides guidance that is directly relevant to AI in public-sector and development contexts. Its Artificial Intelligence Landscape Assessment (AILA) looks at countries across three areas: the AI ecosystem, AI for government, and AI regulation and ethics. It considers factors such as government capacity, accountability, inclusivity, safety, transparency, infrastructure, skills, and data.

These points also relate to the country-context section, where governance requirements need to reflect differences in institutional capacity, connectivity, digital literacy, and regulation. UNDP also emphasizes adapting AI governance to local conditions. I apply that idea at the deployment level here through specific controls such as evaluation gates, live monitoring, human escalation, and post-deployment audits.

The **EU AI Act**, adopted in 2024, takes a different approach by establishing risk categories such as unacceptable, high, and limited risk, with requirements that increase according to the category.

The risk matrix in this document uses a similar idea of matching governance effort to the seriousness of the risk. The purpose here is internal governance for a particular use case alongside applicable legal and regulatory requirements.

The **WHO's Ethics and Governance of AI for Health**, published in 2021, sets out six principles for AI in health, including protecting human autonomy, transparency, and equity.

That guidance also applies directly to the health-related risks discussed above. The requirement for human escalation in red-flag health scenarios reflects the broader principle that AI should support human decision-making rather than replace human judgment where physical safety is involved.

Taken together, these four frameworks provide the broader responsible-AI foundation for this document. The approach here applies that established practice to the specific risks identified in this evaluation.

---

## 5. Implementation Roadmap

### 0–3 months

* Make the rubric-based evaluation harness a mandatory pre-deployment gate for new AI-assisted public-service tools.

* Add the two currently missing areas, bias/consistency and privacy/data handling, before any deployment involving eligibility or entitlement decisions.

* Define the human escalation process and staffing model for each country office before going live.

### 3–6 months

* Run the first full post-deployment audit for existing pilots and establish baseline scores for each criterion and deployment.

* Build the paired demographic-framing test set needed to make bias testing operational.

* Pilot the low-connectivity logging approach in at least one lower-connectivity office. This keeps the monitoring process from being designed only around high-connectivity environments.

### 6–12 months

* Establish a standard audit cadence across live deployments, with quarterly reviews as the default starting point.

* Create an internal comparison across deployments to identify which risks consistently recur and which are specific to particular countries or contexts.

* Expand language and accessibility testing based on actual deployment experience instead of continuing to rely on the current three-language proof of concept.

---

## Limitations

* 16 queries, one model family, single-country prototype; not validated in real production.

* Because the same model family was used for response generation and judging, the scores should be treated as an initial evaluation signal rather than an independent benchmark.

* Likelihood ratings describe what this evaluation measured, not predictions for other model/language/country.

* Bias/privacy are intentionally described as gaps rather than artificial scores; closing these gaps is a prerequisite for real deployment.

* The related frameworks section shows how the approach relates to existing guidance; it is not a complete regulatory review.

---

## Related work

* [Responsible AI Evaluation Harness for Public-Service Chatbots](https://github.com/Harsheen-Kaur-Projects/public-service-ai-evaluation-harness) — the evaluation pipeline and findings that this framework is based on

* [CivicAssist — Public-Service AI Product, Requirements to Prototype](https://github.com/Harsheen-Kaur-Projects/civicassist-product-case-study) — the product-level response to the same problem, including the escalation and uncertainty-handling patterns referenced above