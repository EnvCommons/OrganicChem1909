# OrganicChem1909

[![OpenReward Environment](https://img.shields.io/badge/%E2%AD%90%20OpenReward-Environment-f7e6cc)](https://openreward.ai/GeneralReasoning/OrganicChem1909)

## Description

**OrganicChem1909** is an environment for evaluating agents on organic chemistry questions derived from "Practical Methods of Organic Chemistry" (1909). Questions test procedural understanding and conceptual comprehension of pre-modern organic chemistry practices. An LLM grader evaluates answers for conceptual correctness, accepting alternative nomenclature but grading strictly: only a fully correct answer scores.

## Capabilities

- Answering organic chemistry questions requiring procedural understanding
- Reasoning about chemical mechanisms and principles
- Understanding historical chemistry nomenclature (1909 conventions)
- Demonstrating conceptual comprehension rather than rote recall

## Compute Requirements

OrganicChem1909 does not require a sandbox. It has minimal compute requirements.

## License

[MIT](https://opensource.org/license/mit).

## Tasks

There are three splits: train, validation, and test. Questions are loaded from a parquet file (`organicchem1909_questions.parquet`) and span multiple categories and difficulty levels. Each question includes metadata: chapter, page reference, category, difficulty, and a context snippet from the source textbook.

## Reward Structure

This is a sparse reward environment with a **binary** reward. The agent replies with its response as an ordinary message (no tool call), which ends the rollout, and the environment grades that message text using an LLM grader (gpt-5-mini), which returns one of three verdicts. Only the first earns anything:

- **CORRECT** → reward **1.0**. Every key point of the reference answer is present, with no chemical errors and no material omissions.
- **PARTIALLY_CORRECT** → reward **0.0**. Partially sound reasoning earns nothing; there is no partial credit.
- **INCORRECT** → reward **0.0**. The answer is wrong or missing key information.

An empty answer scores 0.0 without invoking the grader.

The grader also emits a numeric confidence score. It is recorded in the tool
result's `metadata` for analysis but does **not** shape the reward — it is an
uncalibrated free-choice number that varies by roughly ±0.05 when the same
answer is regraded, so the discrete verdict is the stable signal.

Grading rules:
- Both IUPAC names, common names, and historical 1909 terminology are accepted.
- Evaluation focuses on conceptual correctness and understanding, not exact wording.
- Grading is strict: an answer that is incomplete, vague, or partially correct is not marked CORRECT.
- Safety awareness is valued even if not in the reference answer.

## Data

Questions are sourced from "Practical Methods of Organic Chemistry" (1909), a public domain textbook. The dataset is stored as a parquet file on the OpenReward platform.

## Tools

None. The model is given no tools: it answers the chemistry question as an
ordinary message, and that message ends the rollout.

Grading runs through a hidden `@terminal` tool, which sends the message text to
the LLM grader alongside the reference answer and returns the grade and score.
The reference answer and the grader's written feedback are not returned.

## Time Horizon

OrganicChem1909 is a single-turn environment. The agent receives a question and submits one answer. Each task requires exactly one tool call.

## Environment Difficulty

[Statistics on environment difficulty here]

## Other Environment Requirements

OrganicChem1909 requires an OpenAI API key (`OPENAI_API_KEY` secret) for LLM-based grading of answers.

```bash
export OPENAI_API_KEY=your_api_key_here
```

Pass the key via the secrets parameter when creating a session:

```python
async with environment.session(task=task, secrets={"openai_api_key": OPENAI_API_KEY}) as session:
```

## Safety

Agents in OrganicChem1909 are asked to answer chemistry questions from a historical textbook. The environment does not present direct safety risks, as agents only provide text answers with no access to external systems, tools, or the internet.

## Citations

```bibtex
@dataset{GROrganicChem1909,
  author    = {General Reasoning Inc. Team},
  title     = {OrganicChem1909},
  year      = {2026},
  publisher = {OpenReward},
  url       = {https://openreward.ai/GeneralReasoning/OrganicChem1909}
}
```

```bibtex
@book{gattermann1909practical,
  title={Practical Methods of Organic Chemistry},
  author={Gattermann, Ludwig},
  year={1909}
}
```
