# doc-standards

A [Claude Code](https://claude.com/claude-code) skill that makes documentation pass
professional writing standards — and proves it with a lint gate.

Documentation written by an LLM has a smell: synonym rotation, subordinate clauses,
filler, hedging. This skill removes it with the constraints professional documentation
systems already use. The idea started from
[a post by @dcoderio](https://x.com/dcoderio/status/2079560939007819962) about
ASD-STE100, then grew into a five-layer criteria stack.

## The five layers

| Layer | Question it answers | Source standard |
|-------|--------------------|-----------------|
| 1. Structure | Which document am I writing, for whom? | [Diátaxis](https://diataxis.fr) + ISO 24495-1 |
| 2. Controlled language | Is each sentence unambiguous? | [ASD-STE100](https://www.asd-ste100.org) |
| 3. House style | Is it consistent with developer-docs convention? | [Google](https://developers.google.com/style) / [Microsoft](https://learn.microsoft.com/style-guide) style guides |
| 4. AI readability | Can an agent retrieve and use each section alone? | [llms.txt](https://llmstxt.org) + retrieval practice |
| 5. Verification | Can a machine enforce layers 2–4? | [Vale](https://vale.sh) or the bundled checker |

## Install

Copy the `doc-standards` folder into your skills directory:

```bash
cp -r doc-standards ~/.claude/skills/doc-standards
```

Claude Code picks it up automatically. The skill triggers on requests such as
"write docs", "document this", "improve the README", or "the docs smell AI-written".

## What is inside

```
doc-standards/
├── SKILL.md                      # workflow: structure → draft → style → AI pass → gate
├── references/
│   ├── ste100.md                 # ASD-STE100 digest: word, sentence, and safety-text rules
│   ├── style-guides.md           # Google + Microsoft house-style digest
│   ├── ai-readability.md         # self-contained sections, llms.txt format
│   └── vale-setup.md             # Vale install, per-project config, CI
├── scripts/
│   └── check_docs.py             # zero-dependency lint gate
└── evals/
    └── evals.json                # test prompts for the skill-creator eval loop
```

## The lint gate

`check_docs.py` needs only Python 3. It checks sentence length, passive voice,
banned words, terminology drift, heading hierarchy, and relative-link integrity:

```bash
python3 doc-standards/scripts/check_docs.py docs/
```

When you install [Vale](https://vale.sh) and add a `.vale.ini` to the project, the script
defers its style checks to Vale and keeps only the structural checks. The exit code is 1
when any error-level finding exists, so the gate drops into CI as-is.

Terminology drift is the check that catches the strongest LLM tell. When a document uses
"verify", "check", and "confirm" interchangeably, the gate reports it and the skill
prescribes the fix: a terminology table with one approved term per concept.

## Does it work?

An A/B test with two identical agents wrote a README for the same small CLI tool. The
agent with the skill passed 6 of 6 assertions and produced 1 lint finding. The agent
without it passed 3 of 6 and produced 6 findings — it placed the destructive-action
warning after the command it protects, used a positional "as shown above" reference,
and rotated between "argument", "flag", and "option". The eval prompts ship in
`evals/evals.json` so you can rerun the comparison.

This README passes its own gate:

```bash
python3 doc-standards/scripts/check_docs.py README.md
```

## License

[MIT](LICENSE)
