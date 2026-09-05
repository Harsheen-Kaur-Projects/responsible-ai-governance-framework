Author Harsheen Kaur

LinkedIn: https://www.linkedin.com/in/harsheen285/

GitHub: https://github.com/Harsheen-Kaur-Projects

# Responsible AI Governance Framework for Public-Service Deployments

A governance framework for organizations deploying AI-assisted public services —
built from real findings, not hypothetical risks.

**Read the full framework:** [governance-framework.md](./governance-framework.md)

## Why this exists

Most AI governance frameworks are written before there's much evidence of what
actually goes wrong. This one starts from an actual evaluation: my
[Responsible AI Evaluation Harness](https://github.com/Harsheen-Kaur-Projects/public-service-ai-evaluation-harness)
found that a real model's worst failures weren't on the adversarial questions
designed to catch it — two of its three lowest scores came from ordinary,
unflagged social-protection questions. That single finding shapes the entire
risk matrix and monitoring approach in this document.

## How this connects to my other two projects

- **[Evaluation harness](https://github.com/Harsheen-Kaur-Projects/public-service-ai-evaluation-harness)** — measures whether the AI performs reliably
- **[CivicAssist](https://github.com/Harsheen-Kaur-Projects/civicassist-product-case-study)** — decides what the product should do when evidence is or isn't available
- **This framework** — decides how an organization manages that risk once the system is live, at scale, across countries

## What's inside

- A risk matrix scored directly from live evaluation data, with two risks
  (bias, privacy) left honestly unscored where the current evaluation can't
  measure them
- A four-part governance cycle: pre-deployment gate, live monitoring, human
  escalation, post-deployment audit
- Country-context considerations for low-connectivity, low-literacy, and
  varying regulatory environments
- How this relates to existing frameworks — OECD AI Principles, UNDP's own
  Artificial Intelligence Landscape Assessment (AILA), the EU AI Act, and
  WHO's AI-in-health ethics guidance
- A phased implementation roadmap (0–3, 3–6, 6–12 months)

## Limitations

16 queries, one model family, single-country prototype — not validated in
production. Full limitations are listed at the end of the framework document.

## License

This project is licensed under the MIT License.
