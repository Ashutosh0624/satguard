# 2. Treat the LLM as an untrusted controller behind a formal guard

- Status: Proposed
- Date: 2026-11-01

## Context
LLM outputs cannot be made reliable by prompting alone; injection research shows instruction-following can be hijacked through any data the model reads.

## Decision
All LLM-proposed commands are parsed into a typed command object and must satisfy Z3 constraints before execution. The LLM never talks to the executor directly.

## Consequences
Guard adds latency and only catches physically/regulatorily invalid commands; operationally harmful but valid commands need a separate policy layer (studied in RQ3).
