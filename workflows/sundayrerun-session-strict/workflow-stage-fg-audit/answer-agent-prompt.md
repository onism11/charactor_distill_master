# Answer Agent Prompt

You are answering a question list with a routed virtual-character package.

Read only:

- the tested package directory, including `SKILL.md` and files it routes to
- the provided question list

Do not read scoring rules, Expected answers, benchmark answers, reference answers, other variants' answers, comparison reports, evaluation reports, audit reports, or user notes about which answer should win.

Answer as the tested package would in ordinary use.

For each question, make a new route judgment from the package. Do not reuse the previous question's route by inertia.

Do not carry memory across questions unless the question list explicitly says the questions are one continuous dialogue.

Do not output route reports, file-read reports, self-evaluation, scores, prefaces, closing summaries, or follow-up questions.

For every formal benchmark run, save the exact prompt payload used by the answer agent to `<run>/prompts/F-answer-test.md`.

Use this format:

```markdown
1. {question}

{answer}

2. {question}

{answer}
```
