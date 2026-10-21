
# Nexmeal

**Surplus today. Sustenance tomorrow.**

AI-powered surplus food rescue. Nexmeal connects restaurants, hotels, hostels and banquet halls with nearby verified NGOs, using **Strands Decider 2B** to decide whether each donation should be dispatched and **Amazon Bedrock** to explain the decision.

Built for the [AWS Agent Experience Hackathon: Strands Decider](https://builder.aws.com/build/hackathons/4f50957c-1d08-37ab-b11c-97516b9adeb4/agent-experience-hackathon-strands-decider).

- **Live demo:** https://nexmeal-aws-hac.vercel.app/
- **Article:** [link to your Builder Center article]
- **Demo video:** [link to your video]

---

## The problem

Good food is thrown away every night while NGOs nearby struggle to feed people. The blocker is coordination and trust: is it safe to send *this* food to *that* kitchen, right now? A wrong "yes" risks spoiled food reaching vulnerable people. A wrong "no" wastes it.

## How it works

1. **Donor registers surplus**: title, category, quantity, storage requirement, pickup deadline.
2. **Safety gates run**: deterministic rules check each candidate NGO (distance, capacity, storage compatibility, deadline, verification).
3. **Strands Decider 2B decides**: `APPROVE`, `REJECT` or `HUMAN_REVIEW`, with a confidence score.
4. **Amazon Bedrock explains**: Claude 3 Haiku writes the dispatch recommendation and NGO notification.
5. **Humans handle uncertainty**: low-confidence matches go to a coordinator review queue.
6. **Impact is tracked**: kg rescued, meals served, carbon diverted, summarised in an ESG report.

## Architecture

```
Donation + candidate NGO
        |
        v
Deterministic safety gates --fail--> REJECT (model cannot overrule)
        | pass
        v
Strands Decider 2B --> APPROVE / REJECT / HUMAN_REVIEW + confidence
        |
        |-- confidence >= 0.72 and APPROVE --> dispatch
        |-- low confidence / tight deadline / borderline distance --> human review
        v
Amazon Bedrock (Claude 3 Haiku) --> explanation + NGO notification
```

**Design principles**

- **Rules before models.** Hard safety constraints stay deterministic.
- **Decider decides, the LLM explains.** The generative model is never in the safety path.
- **Confidence is a feature.** Below the 0.72 threshold, a human decides.

## The Decider decision pattern

| | |
|---|---|
| **Pattern** | Policy (classify a request before the agent acts), Guardrails flavour |
| **Question** | Given this donation and this recipient, should the agent dispatch the food? |
| **Options** | `APPROVE`, `REJECT`, `HUMAN_REVIEW` |
| **State** | `distance_km`, `capacity_available`, `storage_compatible`, `hours_to_deadline`, `recipient_reliability` |
| **Threshold** | `0.72` confidence, below which the match is escalated |

Reuse it in any agent that must classify a request before acting:

1. Run hard rules first; any failure short-circuits to `REJECT`.
2. Call Decider 2B with compact structured state and the typed question.
3. If `confidence < threshold`, route to a human queue.
4. Only then call the LLM to generate the explanation.

**Code:** [path/to/decider_call.py] <!-- point to the file wrapping your Decider call -->

## Benchmark

Seven test cases (TC-01 to TC-07) cover approval, expired donation, unverified NGO, insufficient capacity, tight deadline, storage mismatch and borderline distance.

| Metric | Baseline | Decider 2B |
|---|---|---|
| Accuracy | [x%] | [x%] |
| Decider latency | n/a | [x ms] |
| Bedrock latency | n/a | [x ms] |

<!-- Fill with measured results from your running backend. -->

## Tech stack

- **Decision engine:** Strands Decider 2B
- **LLM:** Amazon Bedrock (Claude 3 Haiku)
- **Frontend:** [React / Vite], deployed on Vercel
- **Backend:** [framework, e.g. FastAPI], REST API under `/api/v1`
- **Hosting:** [AWS services used]

## Project structure

```
.
├── frontend/        # Web app (builds to dist/)
├── backend/         # API, safety gates, Decider + Bedrock integration
├── benchmarks/      # TC-01 to TC-07 test cases
└── README.md
```

<!-- Adjust to match your repo. -->

## Getting started

### Prerequisites

- Node.js [version]
- Python [version]
- AWS account with Bedrock model access enabled
- Strands Decider 2B access

### Backend

```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # add your keys
uvicorn app.main:app --reload
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Environment variables

| Variable | Description |
|---|---|
| `AWS_REGION` | Region with Bedrock access |
| `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` | AWS credentials |
| `BEDROCK_MODEL_ID` | Claude 3 Haiku model ID |
| `DECIDER_[...]` | Strands Decider configuration |
| `VITE_API_URL` | Backend base URL for the frontend |

<!-- Replace with your real variable names. Never commit secrets. -->

## Roadmap

- Real NGO onboarding and verification
- Courier tracking
- Multi-city routing
- Learning from coordinator overrides to improve the review lane

## Disclaimer

All organizations and donations in the demo are fictional.
